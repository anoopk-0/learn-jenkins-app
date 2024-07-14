pipeline {
    agent any  

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
                  ls -la >> files
                  npm ci
                  npm run build
                  node --version >> versionFile
                  npm --version >> versionFile
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                  echo 'Running the application test'
                  npm run test
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/files, **/versionFile', allowEmptyArchive: true
            junit '**/test-results/*.xml'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
