pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '38fca2d5-7088-4ac2-b102-32699605ad28'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                    # Display directory contents and node/npm versions
                    ls -la
                    node --version
                    npm --version

                    # Install dependencies and sync package-lock.json
                    npm install

                    # Now perform npm ci to ensure the lock file is respected
                    npm ci

                    # Build the project
                    npm run build

                    # Display directory contents again after build
                    ls -la
                '''
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
                        sh '''
                            # Run unit tests
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'  // Publish Jest results
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
                        sh '''
                            # Install serve and serve the build directory
                            npm install serve
                            node_modules/.bin/serve -s build &

                            # Wait for the app to start
                            sleep 10

                            # Run Playwright end-to-end tests
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
                    # Install Netlify CLI for deployment
                    npm install netlify-cli

                    # Check Netlify CLI version
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}
