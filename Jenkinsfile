pipeline {
    agent any

    environment {
        INDEX_FILE_NAME = 'index.html'
        NETLIFY_SITE_ID = '27a5dac5-7760-4ba0-8f87-ccc37def6db9'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        // REACT_APP_VERSION="1.0.$BUILD_ID"
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

        stage("Tests") {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        echo 'Test stage started...'
                        sh '''
                            test -f build/$INDEX_FILE_NAME
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('Local E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                            reuseNode true
                        }
                    }

                    steps {
                        echo 'Local E2E stage started...'
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML(
                                [
                                    allowMissing: false, 
                                    alwaysLinkToLastBuild: false, 
                                    keepAll: false, 
                                    reportDir: 'playwright-report', 
                                    reportFiles: 'index.html', 
                                    reportName: 'Playwright Local HTML Report', 
                                    reportTitles: 'Report', 
                                    useWrapperFileDirectly: true
                                ]
                            )
                        }
                    }
                }
            }
        }


        stage("Staging") {

            environment {
                CI_ENVIRONMENT_URL = "STAGING_URL"
            }

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo deployment has begun...
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo Deploying app to netlify site Id: $NETLIFY_SITE_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-logs.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-logs.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-logs.json)
                    npx playwright test --reporter=html
                '''
            }
        }

        stage("Prod") {

            environment {
               CI_ENVIRONMENT_URL = 'https://melodic-kataifi-6876dd.netlify.app'
            }

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo deployment has begun...
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo Deploying app to netlify site Id: $NETLIFY_SITE_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML(
                        [
                            allowMissing: false, 
                            alwaysLinkToLastBuild: false, 
                            keepAll: false, 
                            reportDir: 'playwright-report', 
                            reportFiles: 'index.html', 
                            reportName: 'Playwright Prod HTML Report', 
                            reportTitles: 'Report', 
                            useWrapperFileDirectly: true
                        ]
                    )
                }
            }
        }
    }
}