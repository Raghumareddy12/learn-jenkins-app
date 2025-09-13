pipeline {
    agent any

    environment {
        JEST_JUNIT_OUTPUT_DIR  = "test-results"
        JEST_JUNIT_OUTPUT_NAME = "jest-results.xml"
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                  echo "Listing workspace..."
                  ls -la
                  echo "Node and npm versions"
                  node --version
                  npm --version
                  echo "Installing dependencies..."
                  npm ci
                  echo "Building app..."
                  npm run build
                  echo "Build complete. Listing build folder..."
                  ls -la
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
                    echo "Installing dependencies..."
                    npm ci
                    npm install serve

                    echo "Serving the build folder in background..."
                    nohup npx serve -s build > serve.log 2>&1 &

                    echo "Running Playwright tests..."
                    # Use reporter config in playwright.config.js
                    npx playwright test --output=test-results
                '''
            }
        }
    }

    post {
        always {
            echo "Publishing test results..."
            junit 'test-results/**/*.xml'
        }
    }
}
