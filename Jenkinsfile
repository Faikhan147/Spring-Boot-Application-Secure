pipeline {
 
    agent any 

    environment {
        IMAGE_NAME = "faisalkhan35/spring-boot-app"
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

        stage('Docker Push') {
            steps {
                sh 'docker push ${IMAGE_NAME}:${TAG}'
            }
        }

        stage('Docker Logout') {
            steps {
                sh 'docker logout'
            }
        }

        stage('AKS Deployment') {
            steps {
                withCredentials([
                    azureServicePrinciple(
                        credentialsId:  'azure-sp',
                        clientIdVariable: 'AZ_CLIENT_ID',
                        clientSecretVariable: 'AZ_CLIENT_SECRET',
                        tenantIdVariable: 'AZ_TENANT_ID',
                        subscriptionIdVariable: 'AZ_SUBSCRIPTION_ID'
                    )
                ])
                    sh '''
                    az login --service-principle -u "$AZ_CLIENT_ID" -p "$AZ_CLIENT_SECRET" --tenant "$AZ_TENANT_ID"
                    az account set --subscription "$AZ_SUBSCRIPTION_ID"
                    az aks get-credentials --resource-group faisal --name javacluster --overwrite-existing
                    kubectl create deployment spring-boot-app --image=${IMAGE_NAME}:${TAG}
                    kubectl expose deployment spring-boot-app --port=80 --target-port=8080 --type=LoadBalancer
                    '''
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
            sh 'echo "Deployment is Successful"'
        }

        failure {
            sh '''
            echo "Deployment is Failed Rollback Initiated"
            kubectl rollout undo deployment/spring-boot-app
            '''
        }
    }
}
