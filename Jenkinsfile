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
            }
        }
    }
}
