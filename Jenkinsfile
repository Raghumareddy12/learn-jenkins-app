pipeline {
    agent any

    environment {
        NODE_IMAGE = 'node:18-alpine'
        PLAYWRIGHT_IMAGE = 'mcr.microsoft.com/playwright:v1.39.0-jammy'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Installing dependencies and building app...'
                script {
                    docker.image(NODE_IMAGE).inside('-u 0:0') {
                        sh '''
                            npm ci --network-timeout=600000
                            npm run build
                        '''
                    }
                }
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Running Jest Unit Tests'
                script {
                    docker.image(NODE_IMAGE).inside('-u 0:0') {
                        sh '''
                            if [ -f build/index.html ]; then
                                echo "✅ build/index.html exists."
                            fi
                            npm test -- --watchAll=false || true
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

        stage('E2E Tests') {
            steps {
                echo 'Running Playwright E2E Tests'
                script {
                    docker.image(PLAYWRIGHT_IMAGE).inside('-u 0:0') {
                        sh '''
                            npm ci --network-timeout=600000 || true
                            mkdir -p test-results
                            npx playwright test || true
                        '''
                    }
                }
            }
            post {
                always {
                    echo 'Publishing E2E test results'
                    junit 'test-results/**/*.xml'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }
    }
}
