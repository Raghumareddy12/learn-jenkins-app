pipeline {
    agent any

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
                echo "Running unit tests"
                sh '''
                    if [ -f build/index.html ]; then
                      echo "✅ build/index.html exists."
                    else
                      echo "❌ build/index.html missing!"
                      exit 1
                    fi
                '''
                sh 'npm test -- --watchAll=false'
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.55.0-jammy'
                    reuseNode true
                }
            }
            steps {
                echo "Running Playwright E2E tests"
                sh '''
                    npm install serve
                    nohup node_modules/.bin/serve -s build &   # serve app in background
                    npx playwright test
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
