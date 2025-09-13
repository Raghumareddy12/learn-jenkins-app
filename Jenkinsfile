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
                  ls -la
                  node --version
                  npm --version
                  npm ci
                  npm run build
                  ls -la
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "Running Unit Tests"
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

        stage('E2E') {
         agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
            }
        steps {
        echo "Running Playwright E2E tests"
        sh '''
            npm ci
            npm install serve
            nohup npx serve -s build > serve.log 2>&1 &

            # Run Playwright tests and output JUnit XML
            npx playwright test --reporter=junit=test-results/playwright-results.xml
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
} // <-- properly closes pipeline
