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
                            node_modules/.bin/serve -s build &
                            sleep 10 
                            npx playwright test --reporter=html
                        '''
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
                        junit 'jest-results/junit.xml'
                  
                    }
                }
            }
        }
    }
}