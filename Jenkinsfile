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

        stage('AKS Deployment') {
            steps {
                withAzureCLI(credentialsId: 'azure-sp') {
                    sh '''
                    az account show
                    az aks get-credentials --resource-group faisal --name javacluster --overwrite-existing
                    kubectl create deployment spring-boot-app --image=${IMAGE_NAME}:${TAG}
                    kubectl expose deployment spring-boot-app --port=80 --target-port=8080 --type=LoadBalancer
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
