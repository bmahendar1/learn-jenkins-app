pipeline {
    agent any

    environment {
        INDEX_FILE_NAME = 'index.html'
        NETLIFY_SITE_ID = '27a5dac5-7760-4ba0-8f87-ccc37def6db9'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION="1.0.$BUILD_ID"
    }
    
    stages {

        stage('Build Docker image') {
            steps {
                sh 'docker build -t custom-playwright-image .'
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
                sh '''
                    npm ci
                    npm run build
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
                            image 'custom-playwright-image'
                            reuseNode true
                        }
                    }

                    steps {
                        echo 'Local E2E stage started...'
                        sh '''
                            #npm install serve
                            serve -s build &
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
                                    reportName: 'Local HTML Report', 
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
                    image 'custom-playwright-image'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo deployment has begun...
                    #npm install netlify-cli node-jq
                    netlify --version
                    echo Deploying app to netlify site Id: $NETLIFY_SITE_ID
                    netlify status
                    netlify deploy --dir=build --json > deploy-logs.json
                    node-jq -r '.deploy_url' deploy-logs.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-logs.json)
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
                            reportName: 'Staging HTML Report', 
                            reportTitles: 'Report', 
                            useWrapperFileDirectly: true
                        ]
                    )
                }
            }
        }

        stage("Prod") {

            environment {
               CI_ENVIRONMENT_URL = 'https://melodic-kataifi-6876dd.netlify.app'
            }

            agent {
                docker {
                    image 'custom-playwright-image'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    echo deployment has begun...
                    #npm install netlify-cli
                    netlify --version
                    echo Deploying app to netlify site Id: $NETLIFY_SITE_ID
                    netlify status
                    netlify deploy --dir=build --prod
                    sleep 10
                    npx playwright test --reporter=html
                    test -f playwright-report/index.html
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
                            reportName: 'Prod HTML Report', 
                            reportTitles: 'Report', 
                            useWrapperFileDirectly: true
                        ]
                    )
                }
            }
        }
    }
}