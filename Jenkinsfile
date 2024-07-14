pipeline {
    agent any  

    environment {
        DOCKER_IMAGE = 'node:18-alpine'
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
                    
                    // Capture directory listing
                    sh 'ls -la > files'

                    // Install dependencies and build the project
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
                                // Check if the build artifact exists
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
