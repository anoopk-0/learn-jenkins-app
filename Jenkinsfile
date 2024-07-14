pipeline {
    agent any  

    stages {
        stage('w/o docker') {  
            steps {
                sh '''
                echo "hello world no docker"
                ls -la >> container-no
                '''
            }
        }
        
        stage('w/ docker') {  
            agent {
                docker {
                    image "node:18-alpine"  
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la >> container-yes
                echo "hello world with docker"
                npm --version  
                '''
            }
        }
    }
}
