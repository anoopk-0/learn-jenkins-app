pipeline {
    agent any  

    environment {
        DOCKER_IMAGE = 'node:18-alpine'
        NETLIFY_SITE_ID = '58fe2438-b69b-401f-90d8-d863d2d98bf4'
        NETLIFY_AUTH_TOKEN = credentials('netify-token')
    }

    stages {
        stage('Docker') {
            steps {
                sh '''
                 docker build -t my-image .
                '''
            }
        }
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
                stage('e2e test') {
                        agent {
                            docker {
                                image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                                reuseNode true
                            }
                        }
                        steps {
                            script {
                            sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & 
                            sleep 10
                            npx playwright test --reporter=line
                            '''
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
        stage('stag e2e test') {
                agent {
                    docker {
                        image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                        reuseNode true
                    }
                }

                environment {
                    CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
                }
                
                steps {
                    script {
                    sh '''
                        npx playwright test --reporter=line
                    '''
                }
            }
            post {
                always {

                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Stag HTML Report', reportTitles: '', useWrapperFileDirectly: true])
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
        stage('Prod e2e test') {
                agent {
                    docker {
                        image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                        reuseNode true
                    }
                }

                environment {
                    CI_ENVIRONMENT_URL = 'https://deft-parfait-2116ef.netlify.app'
                }
                
                steps {
                    script {
                    sh '''
                        npx playwright test --reporter=line
                    '''
                }
            }
        }
    }
    post {
        always {
            // Archive build artifacts and test results
            archiveArtifacts artifacts: '**/files, **/versionFile', allowEmptyArchive: true

            // For reporting about the test
            junit 'jest-results/*.xml'

            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
