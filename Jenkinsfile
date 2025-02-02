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
                        npx playwright test
                    '''
                }
            }
        }
    }
    post {
        always {
            // Only attempt to record test results if a test report exists
            script {
                def testResults = 'test-results/junit.xml'
                if (fileExists(testResults)) {
                    junit testResults
                } else {
                    echo "No test results found. Skipping JUnit report."
                }
            }
        }
    }
}