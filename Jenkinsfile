pipeline {
    agent any

    environment {
        IMAGE_TAG = "1.0.3"
        IMAGE_NAME = "console-mqtt-batch"
        ACR_NAME = "acrmqtt"
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        RESOURCE_GROUP = "myResourceGroupAppNode2"
        CONTAINER_NAME = "console-mqtt-batch-container"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Obtener el código fuente desde tu repositorio
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Construir la imagen Docker localmente
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to Azure') {
            steps {
                script {
                    withCredentials([azureServicePrincipal('sp-azure-curso-devops')]) {
                        // Autenticarse en Azure usando un principal de servicio
                        sh "az account clear"
                        sh "az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}"
                    }
                }
            }
        }

        stage('Login to ACR') {
            steps {
                script {
                    // Obtener el token de autenticación para ACR
                    sh "az acr login --name ${ACR_NAME}"
                }
            }
        }

        stage('Tag Docker Image') {
            steps {
                script {
                    // Etiquetar la imagen Docker para el registro de ACR
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Push to ACR') {
            steps {
                script {
                    // Subir la imagen Docker a ACR
                    sh "docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Azure Container Instances') {
            steps {
                script {
                    // Crear y desplegar el contenedor en Azure Container Instances
                    sh "az container delete --resource-group ${RESOURCE_GROUP} --name ${CONTAINER_NAME}"
                    sh "az container create --resource-group ${RESOURCE_GROUP} --name ${CONTAINER_NAME} --image ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${IMAGE_TAG} --registry-login-server ${ACR_LOGIN_SERVER} --registry-username \$(az acr credential show --name ${ACR_NAME} --query username -o tsv) --registry-password \$(az acr credential show --name ${ACR_NAME} --query passwords[0].value -o tsv) --cpu 1 --memory 1 --os-type Linux"
                }
            }
        }
        
    }
}