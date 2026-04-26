pipeline {
    agent any

    environment {
        IMAGE_NAME = "dipanshubisen/lbimage"
        TAG = "${BUILD_NUMBER}"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "my-cluster"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/DipanshuBisen/aws-lb-cicd-setup.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME:$TAG ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds',
                    usernameVariable: 'USER', passwordVariable: 'PASS')]) {

                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $IMAGE_NAME:$TAG
                    """
                }
            }
        }

        stage('Update Kubeconfig') {
            steps {
                sh """
                aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                kubectl set image deployment/my-app my-app=$IMAGE_NAME:$TAG
                kubectl rollout status deployment/my-app
                """
            }
        }
    }
}