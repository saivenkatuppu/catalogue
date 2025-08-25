pipeline {
    agent  {
        label 'AGENT-1'
    }
    environment { 
            appVersion= ''
            REGION = "us-east-1"
            ACC_ID = "990063103412"
            PROJECT = "roboshop"
            COMPONENT = "catalogue" 
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
     parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    } 
    // Build
    stages {
        stage('Read Package.json') {
            steps {
                 script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Package version: ${appVersion}"
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script{
                    sh """
                    npm install 

                    """
                }
            }
        }
         stage('Unit Testing') {
            steps {
                script {
                   sh """
                        echo "unit tests"
                   """
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    withAWS(credentials: 'aws-auth', region: 'us-east-1') {
                        sh """
                          aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                          docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                          docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }
        stage('trigger deploy') {
             when{
                expression { params.deploy }
            steps {
                script {
                    build job: 'catalogue-cd',
                    parameters: [
                         string(name: 'appVersion', value: "${appVersion}"),
                         string(name: 'deploy_to', value: 'dev')
                     ],
                    propagate: false,
                    wait: false
                    }
                }
            }
        }
        
    }

    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
        success { 
            echo 'Hello Success'
        }
        failure { 
            echo 'Hello Failure'
        }
    }
} 