pipeline {
    agent any

    environment {
        INDEX_FILE_NAME = 'index.html'
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

        stage('Test') {
            steps {
                echo 'Test stage started...'
                sh '''
                    test -f build/$INDEX_FILE_NAME
                    npm text
                '''
            }
        }
    }
}