pipeline {
    agent any 

    environment {
        NETLIFY_SITE_ID = 'b0d59fd2-d7c2-4055-994a-b037cfbf5e46'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        FILE_NAME = 'index.html'
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
                        echo 'Unit Test stage starts'
                        sh '''
                            #test -f build/$FILE_NAME
                            npm test
                        '''
                        echo 'Unit Test stage ends'
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                            npm install serve
                            node_modules/.bin/serve -s build & 
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                        echo 'End2End stage ends'
                    }
                    post {
                        always {
                            publishHTML(target:[
                                allowMissing: false, 
                                alwaysLinkToLastBuild: 
                                false, icon: '', 
                                keepAll: true, 
                                reportDir: 'playwright-report', 
                                reportFiles: 'index.html', 
                                reportName: 'Playwright Local', 
                                reportTitles: '', 
                                useWrapperFileDirectly: true
                            ])
                        }
                    }                    
                }
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@21.6.0
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --no-build --message "Deployment"
                '''
            }
        }       

        stage('Approval') {
            steps {
                echo 'Approval stage begins'
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
                echo 'Approval step ends'
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@21.6.0
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod --no-build --message "Deployment"
                '''
            }
        }
        
        stage('Prod End2End') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true 
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://creative-kulfi-b6afaa.netlify.app/'
            }
            
            steps {
                echo 'End2End stage starts'
                sh '''
                    npx playwright test --reporter=html
                '''
                echo 'End2End stage ends'
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false, 
                        alwaysLinkToLastBuild: false, 
                        icon: '', 
                        keepAll: false, 
                        reportDir: 'playwright-report', 
                        reportFiles: 'index.html', 
                        reportName: 'Playwright End2End', 
                        reportTitles: '', 
                        useWrapperFileDirectly: true
                    ])
                }
            }                    
        }        
        
    }

}