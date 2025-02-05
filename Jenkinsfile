pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '38fca2d5-7088-4ac2-b102-32699605ad28'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI_ENVIRONMENT_URL = 'https://rainbow-mooncake-99927d.netlify.app'  // Default fallback environment URL
        REACT_APP_VERSION = "1.0.$BUILD_ID"  // Add version for tagging or deployment if needed
    }

    stages {

        stage('Setup dependencies') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                script {
                    try {
                        sh '''
                            npm install netlify-cli node-jq
                            node_modules/.bin/netlify --version
                            node_modules/.bin/node-jq --version
                        '''
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
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
                script {
                    try {
                        sh '''
                            ls -la
                            node --version
                            npm --version
                            npm install
                            npm run build
                            ls -la
                        '''
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
            post {
                always {
                    archiveArtifacts allowEmptyArchive: true, artifacts: '**/build/**/*', onlyIfSuccessful: false
                }
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        script {
                            try {
                                sh 'npm test'
                            } catch (Exception e) {
                                currentBuild.result = 'FAILURE'
                                throw e
                            }
                        }
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
                            image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                            reuseNode true
                        }
                    }

                    steps {
                        script {
                            try {
                                sh '''
                                    npm install serve
                                    node_modules/.bin/serve -s build &  # Serve the build folder
                                    sleep 10
                                    npx playwright test --reporter=html
                                '''
                            } catch (Exception e) {
                                currentBuild.result = 'FAILURE'
                                throw e
                            }
                        }
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, 
                                          reportDir: 'playwright-report', reportFiles: 'index.html', 
                                          reportName: 'Local E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                }
            }
            
            environment {
                CI_ENVIRONMENT_URL = 'StAGING_URL_TO_BE_SET'
            }

            steps {
                script {
                    try {
                        echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                        sh 'node_modules/.bin/netlify status'
                        sh 'node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json'
                        CI_ENVIRONMENT_URL = sh(script: 'node_modules/.bin/node-jq -r ".deploy_url" deploy-output.json', returnStdout: true).trim()
                        echo "Staging deployed at $CI_ENVIRONMENT_URL"
                        npx playwright test --reporter=html
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, 
                                 reportDir: 'playwright-report', reportFiles: 'index.html', 
                                 reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.50.1-noble'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = "${env.CI_ENVIRONMENT_URL}"  // Dynamic production URL
            }

            steps {
                script {
                    try {
                        echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                        sh 'node_modules/.bin/netlify status'
                        sh 'node_modules/.bin/netlify deploy --dir=build --prod --json > prod-deploy-output.json'
                        CI_ENVIRONMENT_URL = sh(script: 'node_modules/.bin/node-jq -r ".deploy_url" prod-deploy-output.json', returnStdout: true).trim()
                        echo "Production deployed at $CI_ENVIRONMENT_URL"
                        npx playwright test --reporter=html
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, 
                                 reportDir: 'playwright-report', reportFiles: 'index.html', 
                                 reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }

    post {
        success {
            // Slack Notification on Success
            slackSend channel: '#your-channel', color: 'good', message: "Build #${env.BUILD_NUMBER} was successful!"
        }
        failure {
            // Slack Notification on Failure
            slackSend channel: '#your-channel', color: 'danger', message: "Build #${env.BUILD_NUMBER} failed!"
        }
        always {
            // Clean up any remaining resources or steps to run
            echo 'Pipeline completed!'
        }
    }
}
