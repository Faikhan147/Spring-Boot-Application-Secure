pipeline {

    agent any

    environment {
        IMAGE_NAME = "faisalkhan35/spring-boot-app:latest"
        NEXUS_CREDS = credentials('nexus-creds')
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
    }

    tools {
        maven 'Maven17'
    }

    stages {

        stage('Git Clone') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar-scanners') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Nexus upload') {
            steps {
                sh 'curl -v -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW --upload-file target/Spring-Boot-application.-0.0.1-SNAPSHOT.jar http://4.240.107.3:8081/repository/maven-snapshots/com/example/Spring-Boot-application./0.0.1-SNAPSHOT/Spring-Boot-application.-0.0.1-SNAPSHOT.jar'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }


        stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
            }
        }

        stage('Docker Hub Push') {
            steps {
                sh 'docker push ${IMAGE_NAME}'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Checking Application Health Check"
                curl --fail http://135.13.176.227 || exit 1
                sleep 15
                echo "Application is Healthy"
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
