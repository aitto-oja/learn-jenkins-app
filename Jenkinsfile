pipeline {
    agent any 

    environment {
        FILE_NAME = 'index.html'
    }

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
        */
        
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true 
                }
            }
            steps {
                echo 'Test stage start'
                sh '''
                    #test -f build/$FILE_NAME
                    npm test
                '''
                echo 'Test stage start'
            }
        }

        stage('End2End') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                }
            }
            steps {
                echo 'End2End stage starts'
                sh '''
                    npm install -g serve
                    serve -s build
                    npx playwright test
                '''
                echo 'End2End stage ends'
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}