pipeline{
    agent any 
    stages {
        stage('build'){
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode True
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
                            reuseNode True
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build
                            npx playwright test
                        '''
                    }
                }
                stage ('test'){
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode True
                        }
                    }
                    steps {
                        sh '''
                        npm test 
                        '''
                    }
                }
            }
        }
    }
}