pipeline{
    agent any 
    stages {
        stage('build'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm ci 
                    npm run build 
                '''
            }
        }
        stage('test'){
            parallel {
                stage ('E2E'){
                    agent{
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.55.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        
                    }
                }
                stage ('test'){
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
                            junit 'test-results/junit.xml'
                            publishHTML([allowmissing :false, alwaysLinkToLastBuild :false, keepAll :true, reportDir :'test-results/html', reportFiles :'index.html', reportName :'HTML Report'])
                        }
                    }
                }
            }
        }
    }
}