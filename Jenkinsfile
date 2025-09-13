pipeline {
    agent any

    environment {
        JEST_JUNIT_OUTPUT_DIR  = "test-results"
        JEST_JUNIT_OUTPUT_NAME = "jest-results.xml"
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

                    # Install jest-junit reporter if missing
                    npm install --save-dev jest-junit

                    # Run Jest with JUnit reporter
                    npm test -- --watchAll=false --testResultsProcessor=jest-junit
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

                    # Run Playwright tests with JUnit reporter
                    npx playwright test --reporter=junit

                    # Kill serve after tests
                    pkill -f "npx serve"
                '''
            }
        }
    }

    post {
        always {
            echo "Publishing test results..."
            junit 'test-results/**/*.xml'
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
