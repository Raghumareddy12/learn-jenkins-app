pipeline {
    agent any

    environment {
        JEST_JUNIT_OUTPUT_DIR  = "test-results"
        JEST_JUNIT_OUTPUT_NAME = "junit.xml"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "Installing dependencies and building app..."
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la build
                '''
            }
        }

        stage('Unit Tests') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "Running Jest Unit Tests"
                sh '''
                    if [ -f build/index.html ]; then
                        echo "✅ build/index.html exists."
                    else
                        echo "❌ build/index.html missing!"
                        exit 1
                    fi

                    # Run Jest (jest-junit is already configured in package.json)
                    npm test -- --watchAll=false
                '''
            }
        }

        stage('E2E Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                echo "Running Playwright E2E Tests"
                sh '''
                    npm ci
                    npm install serve
                    mkdir -p test-results

                    # Serve build folder in background
                    nohup npx serve -s build > serve.log 2>&1 &

                    # Run Playwright tests
                    npx playwright test 

                    # Kill serve after tests
                    pkill -f "npx serve"
                '''
            }
        }
    }

    post {
    always {
        script {
            if (fileExists('test-results/junit.xml') || fileExists('test-results/playwright-results.xml')) {
                junit 'test-results/**/*.xml'
            } else {
                echo "No test results found, skipping junit step"
            }
        }
    }
}
}
