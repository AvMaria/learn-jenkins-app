pipeline {
    agent any

    stages {
        stage('Build') {
            agent{
                docker{
                    image'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh'''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                ls -la
                grep ".*" filename > build/index.html
                '''
            }
            
        }
        stage('Test') {
            agent{
                docker{
                    image'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh'''
                npm test
                echo "a"
                '''
            }
        }
    }
}