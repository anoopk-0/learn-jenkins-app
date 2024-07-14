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
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                  echo 'Running the application test'
                  test -f build/index.html
                  npm test
                '''
            }
        }
    }
}
