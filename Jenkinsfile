pipeline {

    agent any

    tools {
        maven 'maven3'
        jdk 'jdk11'
    }

    environment {

        APP_NAME = "sample-java-app"

        AWS_ACCOUNT = "123456789012"
        AWS_REGION = "us-east-1"

        ECR_REPO = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${APP_NAME}"

        IMAGE_TAG = "${BUILD_NUMBER}"

    }

    stages {

        stage('Checkout Code') {

            steps {

                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/username/sample-java-app.git'

            }

        }

        stage('SonarQube Scan') {

            environment {
                SONAR_TOKEN = credentials('sonar-token')
            }

            steps {

                sh '''
                mvn sonar:sonar \
                -Dsonar.login=$SONAR_TOKEN
                '''

            }

        }

        stage('Build Docker Image') {

            steps {

                sh """
                docker build -t $ECR_REPO:$IMAGE_TAG .
                """

            }

        }

        stage('Trivy Image Scan') {

            steps {

                sh """
                trivy image $ECR_REPO:$IMAGE_TAG
                """

            }

        }

        stage('Login to AWS ECR') {

            steps {

                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh """
                    aws ecr get-login-password \
                    --region $AWS_REGION \
                    | docker login \
                    --username AWS \
                    --password-stdin $AWS_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com
                    """

                }

            }

        }

        stage('Push Image to ECR') {

            steps {

                sh """
                docker push $ECR_REPO:$IMAGE_TAG
                """

            }

        }

    }

}
