pipeline {
    agent any

    stages {
        /*
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    // Install dependencies first
                    sh '''
                        ls -la
                        node --version
                        npm --version
                        npm ci
                        npm run build
                    '''
                }
            }
        }
        */
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    // Install dependencies (ensure react-scripts are installed)
                    sh '''
                        npm install
                        npm test
                    '''
                }
            }
        }
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                }
            }
            steps {
                script {
                    sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &  # Change backslash to forward slash
                        sleep 10
                        npx playwright install
                        npx playwright test
                    '''
                }
            }
        }
    }
    post {
        always {
           junit 'jest-results/junit.xml'
        }
    }
}