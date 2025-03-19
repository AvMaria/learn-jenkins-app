pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '40b8e94f-254a-4999-ad02-e5e48845203b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')  
        REACT_APP_VERSION = "1.0.$BUILD_ID"    
    }

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
                '''
            }
            
        }        
        stage ('Testing'){
            parallel{
                stage('Unit Test') {        
                    agent{
                        docker{
                            image'node:18-alpine'
                            reuseNode true
                        }
                    }    
                    steps {
                        sh'''
                        test -f build/index.html
                        npm test
                        echo "a"
                        '''
                    }
                    post {
                        always{
                            junit 'jest-results/junit.xml'                            
                        }
                    }
                }
                stage('Test E2E') {        
                    agent{
                        docker{
                            image'my-playwright'
                            reuseNode true
                        }
                    }    
                    steps {
                        sh'''
                        serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        echo "TestE2E completed"
                        '''
                    }
                    post {
                        always{                           
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Test E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }    

        stage('Deploy staging') {        
                    agent{
                        docker{
                            image'my-playwright'
                            reuseNode true
                        }
                    }                   
                    steps {
                        sh'''
                        node --version 
                        netlify --version
                        echo "Deploying to Staging site ID: $NETLIFY_SITE_ID"
                        netlify status 
                        netlify deploy --dir=build --json > deploy-output.json
                        STAGING_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                        export CI_ENVIRONMENT_URL="$STAGING_URL"                        
                        echo "Executing e2e..."                           
                        npx playwright test --reporter=html
                        echo "Stage E2E completed"
                        '''
                    }
                    post {
                        always{                           
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Stage E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
        }           
        
        stage('Deploy Prod') {        
                    agent{
                        docker{
                            image'my-playwright'
                            reuseNode true
                        }
                    }
                    environment {                        
                        CI_ENVIRONMENT_URL = 'https://fancy-speculoos-05ae24.netlify.app'
                    }    
                    steps {
                        sh'''       
                        node --version
                        npm install netlify-cli 
                        netlify --version 
                        echo "Deploying to Production site ID: $NETLIFY_SITE_ID"
                        netlify status 
                        netlify deploy --dir=build --prod   
                        echo "Starting E2E execution"              
                        npx playwright test --reporter=html
                        echo "Prod E2E completed"
                        '''
                    }
                    post {
                        always{                           
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
        }
    
    }
    
}