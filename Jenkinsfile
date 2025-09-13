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
        stage ('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "Test stage"
                sh '''
                    if [ -f build/index.html ]; then
                    echo "build/index.html exists."

                    else
                    echo "build/index.html missing!"
                    exit 1
                    fi

                '''
                 // Run unit tests
                sh 'npm test -- --watchAll=false'
            }
        }
        stage ('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.55.0-noble'
                    reuseNode true
                }
            }
            steps {
                echo "Test stage"
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build
                    npx playwright test

                '''
                 // Run unit tests
                sh 'npm test -- --watchAll=false'
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}

