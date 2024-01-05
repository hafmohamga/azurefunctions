pipeline {
    agent any
    tools {
        terraform "terraform"
    }
    environment {
        HTTP_TRIGGER = "httptriggerfuncxxxx"  
        RES_GROUP = "rg_abdel_proc" 
        APIM_NAME = "apimvterramodules"
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*/httpTrigger']], extensions: [], userRemoteConfigs: [[credentialsId: 'GitHubcredentials', url: 'https://github.com/Selmouni-Abdelilah/AzureFunctions']])
                     // checkout scmGit(branches: [[name: '*/httpTrigger']], extensions: [], userRemoteConfigs: [[credentialsId: 'hafeez-jenkins-token', url: 'https://github.com/hafmohamga/azurefunctions']])
                }
            }
        }
        stage('Azure login'){
            steps{
                withCredentials([azureServicePrincipal('Azure_credentials')]) {
                    sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
                }
            }
        }
        stage('Terraform ') {
            steps {
                script {
                    dir('Terraform') {
                            sh 'terraform init -upgrade'
                            sh "terraform apply --auto-approve -var 'rg_name=${env.RES_GROUP}' -var 'function_name=${env.HTTP_TRIGGER}' -var 'apim_name=${env.APIM_NAME}' "   

                    }
            }
            }
        }    
    
        stage('Deploy Http trigger Function') {
            steps {
                script { 
                   dir('httpTrigger') {
                        sh 'python3 -m pip install -r requirements.txt'
                        sh 'zip -r  http.zip ./*'
                        sh "az functionapp deployment source config-zip -g ${env.RES_GROUP} -n ${env.HTTP_TRIGGER} --src http.zip"                                   
                    }
                }
            }

        }
        stage('API import'){
            steps {
                script {
                    sh "sed -i 's~url:~url: https://${APIM_NAME}.azure-api.net~' openapi.yaml"
                    sh '''
                    az apim api import \
                        --path "/" \
                        --resource-group ${RES_GROUP} \
                        --service-name ${APIM_NAME} \
                        --api-id ${HTTP_TRIGGER} \
                        --api-type http \
                        --display-name "APIM Function App" \
                        --protocols https \
                        --service-url "https://${HTTP_TRIGGER}.azurewebsites.net/api/http_trigger" \
                        --specification-format OpenApi \
                        --specification-path "openapi.yaml" \
                        --subscription-required false
                    '''
                }
            }
        }
    }
}
