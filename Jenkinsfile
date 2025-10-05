pipeline {
    agent any
    environment {
        NETLIFY_PROJECT_ID = '2a717309-0323-42e5-af74-91842593c212'
        NETLIFY_AUTH_TOKEN = credentials('netlify')
    }
    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    # npm ci
                    # npm run build
                '''
            }
        }

        stage('test') {
            parallel {
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.55.1-jammy' 
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #npm install serve
                            node_modules/.bin/serve -s build &
                            SERVER_PID=$!
                            sleep 10
                            npx playwright test --reporter=html
                            kill $SERVER_PID
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

                stage('unit-test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
            }
        }
        stage('deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    #npm install netlify-cli@20.1.1
                    #node_modules/.bin/netlify --version
                    node_modules/.bin/netlify link --id $NETLIFY_PROJECT_ID
                    #node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    '''
            }
        }
        stage('E2E prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.55.1-jammy' 
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://manojkumardevops.netlify.app/'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright prod', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
