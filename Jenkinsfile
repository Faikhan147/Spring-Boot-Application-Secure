pipeline {
   
    agent any

    environment {
        IMAGE_NAME = "faisalkhan35/my-java-app"
        TAG = "${BUILD_NUMBER}"
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
    }

    tools {
        maven 'Maven17'
    }

    stages {

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${TAG} .'
            }
        }

        stage('DockerHub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
            }
        }

        stage('DockerHub Push') {
            steps {
                sh 'docker push ${IMAGE_NAME}:${TAG}'
            }
        }

        stage('DockerHub Logout') {
            steps {
                sh 'docker logout'
            }
        }

        stage('Deploy to AKS') {
            steps {
                withCredentials([
                    azureServicePrincipal(
                        credentialsId: 'azure-sp',
                        clientIdVariable: 'AZ_CLIENT_ID',
                        clientSecretVariable: 'AZ_CLIENT_SECRET',
                        tenantIdVariable: 'AZ_TENANT_ID',
                        subscriptionIdVariable: 'AZ_SUBSCRIPTION'
                    )
                ]) {
                    sh '''
                        az login \
                          --service-principal \
                          --username "$AZ_CLIENT_ID" \
                          --password "$AZ_CLIENT_SECRET" \
                          --tenant "$AZ_TENANT_ID"

                        az account set \
                          --subscription "$AZ_SUBSCRIPTION"

                        az aks get-credentials \
                          --resource-group my-rg \
                          --name myaks

                        kubectl apply -f deployment.yaml
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Checking Application Health Check"
                sleep 15
                curl --fail http://myapp || exit 1
                echo "Application is Alive Now"
                '''
            }
        }
    }

    post {
 
        success {
            sh 'echo "Deployment is Sucessful"'
        }

        failure {
            sh '''
            echo "Application Health Check is Failed Rollback Initiated"
            kubectl rollout undo deployment/spring-boot-app
            '''
        }
    }
}
