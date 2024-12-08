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

        stage('Test') {
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

        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.0-noble'
                    reuseNode true
                }
            }

            steps {
                echo 'E2E stage started...'
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build
                    npx playwright test
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