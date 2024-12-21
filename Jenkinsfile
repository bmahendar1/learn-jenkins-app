pipeline {
    agent any

    environment {
        INDEX_FILE_NAME = 'index.html'
        NETLIFY_SITE_ID = '27a5dac5-7760-4ba0-8f87-ccc37def6db9'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    
    stages {
        // This is single line comment
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # This is to comment a line in this kind of block
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        /*
        * This is multiline comment
        */

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

                stage('Local E2e Tests') {
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


        stage("Deploy(Staging)") {
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
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage("Approval") {
            steps {
                timeout(time:1, unit: 'MINUTES') {
                    input message: 'Are you sure you want to deploy?', ok: 'Yes, Im sure.'
                }
            }
        }

        stage("Deploy(Prod)") {
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
                '''
            }
        }


        stage('Prod E2e Tests') {

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
                echo 'Prod E2E stage started...'
                sh '''
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

        // stage('Test') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         echo 'Test stage started...'
        //         sh '''
        //             test -f build/$INDEX_FILE_NAME
        //             npm test
        //         '''
        //     }
        // }

        // stage('e2e') {
        //     agent {
        //         docker {
        //             image 'mcr.microsoft.com/playwright:v1.49.1-noble'
        //             reuseNode true
        //         }
        //     }

        //     steps {
        //         echo 'E2E stage started...'
        //         sh '''
        //             npm install serve
        //             node_modules/.bin/serve -s build &
        //             sleep 10
        //             npx playwright test --reporter=html
        //         '''
        //     }
        // }
    }

    // post {
    //     always {
    //         junit 'jest-results/junit.xml'
    //         publishHTML(
    //             [
    //                 allowMissing: false, 
    //                 alwaysLinkToLastBuild: false, 
    //                 keepAll: false, 
    //                 reportDir: 'playwright-report', 
    //                 reportFiles: 'index.html', 
    //                 reportName: 'Playwright HTML Report', 
    //                 reportTitles: 'Report', 
    //                 useWrapperFileDirectly: true
    //             ]
    //         )
    //     }
    // }
}