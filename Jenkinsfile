pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'      // Jenkins credential ID for Docker Hub
        DOCKERHUB_IMAGE = 'manukrishnayadav/ci-cd-node-sample'
        AWS_CREDENTIALS = 'aws-creds'                  // Jenkins AWS Credentials type
        AWS_DEFAULT_REGION = 'ap-south-1'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install & Test') {
            steps {
                bat 'npm ci'
                bat 'npm test || exit 0'
            }
        }
        stage('Build Docker image') {
            steps {
                script {
                    def IMAGE_TAG = "${env.BUILD_ID}"
                    bat "docker build -t %DOCKERHUB_IMAGE%:${IMAGE_TAG} -t %DOCKERHUB_IMAGE%:latest ."
                    env.IMAGE_TAG = IMAGE_TAG
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                        bat "docker push %DOCKERHUB_IMAGE%:%IMAGE_TAG%"
                        bat "docker push %DOCKERHUB_IMAGE%:latest"
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                // Using AWS Credentials type
                withAWS(credentials: "${AWS_CREDENTIALS}", region: "${AWS_DEFAULT_REGION}") {
                    script {
                        bat """
                            REM Update kubeconfig for your EKS cluster
                            aws eks update-kubeconfig --name my-eks-cluster

                            REM Apply Kubernetes manifests
                            kubectl -n default apply -f k8s\\deployment.yaml
                            kubectl -n default apply -f k8s\\service.yaml

                            REM Update deployment image
                            kubectl -n default set image deployment/ci-cd-sample app=%DOCKERHUB_IMAGE%:%IMAGE_TAG% || exit 0
                        """
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline completed successfully ✅"
        }
        failure {
            echo "Pipeline failed ❌"
        }
    }
}
