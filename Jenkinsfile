pipeline {
    agent any

    environment {
        INDEX_FILE_NAME = 'index.html'
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
                }

                stage('E2e Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                            reuseNode true
                        }
                    }

                    steps {
                        echo 'E2E stage started...'
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
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

    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML(
                [
                    allowMissing: false, 
                    alwaysLinkToLastBuild: false, 
                    keepAll: false, 
                    reportDir: 'playwright-report', 
                    reportFiles: 'index.html', 
                    reportName: 'Playwright HTML Report', 
                    reportTitles: 'Report', 
                    useWrapperFileDirectly: true
                ]
            )
        }
    }
}