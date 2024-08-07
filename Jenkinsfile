pipeline {
    agent any  

    environment {
        DOCKER_IMAGE = 'node:18-alpine'
        NETLIFY_SITE_ID = '58fe2438-b69b-401f-90d8-d863d2d98bf4'
        NETLIFY_AUTH_TOKEN = credentials('netify-token')
    }

    stages {
         
        stage('Build') {
            agent {
                docker {
                    image "${DOCKER_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                script {
                    // Capture versions of Node and npm
                    sh '''
                      echo "Node Version:" > versionFile
                      node --version >> versionFile
                      echo "npm Version:" >> versionFile
                      npm --version >> versionFile
                    '''
                    

                    sh 'ls -la > files'


                    sh '''
                      npm ci
                      npm run build
                    '''
                }
            }
        }
         stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-jenkins-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws s3 sync build s3://learn-lenkins-app-2024
                    '''
                }
            }
        }
        stage('Test Block Stage'){
            parallel {
                stage('Test') {
                        agent {
                            docker {
                                image "${DOCKER_IMAGE}"
                                reuseNode true
                            }
                        }
                        steps {
                            script {
                                
                                sh '''
                                if [ ! -f build/index.html ]; then
                                    echo "Build artifact not found!"
                                    exit 1
                                fi
                                '''

                                // Run the tests
                                sh 'npm test'
                            }
                        }
                }
                
            }
        }
        stage('deploy stage') {
            agent {
                docker {
                    image "${DOCKER_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                script {
                    sh '''
                      npm install netlify-cli node-jq

                      echo "netlify Version:" > versionFile
                      node_modules/.bin/netlify --version >> versionFile

                      echo 'Deploying to Netlify with site id: ${NETLIFY_SITE_ID}'

                      node_modules/.bin/netlify  status
                      node_modules/.bin/netlify  deploy --dir=build --json > deploy-output.json
                      node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                      
                    '''

                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
            }
        }
        
        stage('deploy prod') {
            agent {
                docker {
                    image "${DOCKER_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                script {
                    sh '''
                      npm install netlify-cli 

                      echo "netlify Version:" > versionFile
                      node_modules/.bin/netlify --version >> versionFile

                      echo 'Deploying to Netlify with site id: ${NETLIFY_SITE_ID}'

                      node_modules/.bin/netlify  status
                      node_modules/.bin/netlify  deploy --dir=build --prod
                    '''
                }
            }
        }
        
    }
    post {
       
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
