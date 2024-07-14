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
                  cleanWs() 
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
                  test -f build/index.html
                 
                '''
            }
        }
    }
}
