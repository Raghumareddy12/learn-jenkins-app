pipeline {
    agent any

    environment {
        NETLIFY_Project_ID = 'a896324c-6076-4960-a7cc-7aaa67476c9f'
        NETLIFY_AUTH_TOKEN = credentials ('netlify-token')
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
        
        stage ('tests') {
            parallel {
                
                stage('unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                             #test -f build/index.html
                             npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                        allowMissing: false,
                                        alwaysLinkToLastBuild: false,
                                        keepAll: true,
                                        reportDir: 'playwright-report',
                                        reportFiles: 'index.html',
                                        reportName: 'Playwright HTML Report',
                                        useWrapperFileDirectly: true
                                        ])
                        }
                    }
                }
            }
        }
                stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production.Project ID:$NETLIFY_Project_ID"
                    node_modules/.bin/netlify status
                '''
            }
        }
    }
}
