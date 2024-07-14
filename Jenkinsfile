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
                  npm install
                  npm run build
                  node --version >> versionFile
                  npm --version >> versionFile
                '''
            }
        }
    }
}
