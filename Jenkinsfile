pipeline {
    agent any

    environment {
       /* NETLIFY_SITE_ID = '40b8e94f-254a-4999-ad02-e5e48845203b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')  
        */
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = "ap-southeast-2"
        AWS_ECS_CLUSTER = "jenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICES_PROD = "JenfinsApp-service-Prod"
        AWS_TASK_DEFINITION = "JenfinsApp-TaskDefinition-Prod"
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
        stage ('build docker image') {
            agent{
                docker{
                    image 'amazon/aws-cli:2.24.27'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                    reuseNode true
                }
            }
           
            steps{
                sh'''
                    amazon-linux-extras install docker
                    docker build -t myjenkinsapp .'''
            }
        }

        stage ('deploy prod aws') {
            agent{
                docker{
                    image 'amazon/aws-cli:2.24.27'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }
           
            steps{
                withCredentials([usernamePassword(credentialsId: 'maria-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh'''
                        aws --version     
                        yum install jq -y                  
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        aws ecs update-service \
                        --cluster $AWS_ECS_CLUSTER \
                        --service $AWS_ECS_SERVICES_PROD \
                        --task-definition $AWS_TASK_DEFINITION:$LATEST_TD_REVISION
                        aws ecs wait services-stable \
                        --cluster $AWS_ECS_CLUSTER \
                        --services $AWS_ECS_SERVICES_PROD
                    '''
                }
                
            }
        } 
        /*  below steps are deprecated since i am not longer using netlify

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
                        STAGING_URL=$(jq -r '.deploy_url' deploy-output.json)
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
        */
    
    }
    
}