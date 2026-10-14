pipeline {
    agent any

    environment {
        ECR_REGISTRY = "342560106932.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE_NAME   = "asg"
        AWS_REGION   = "us-east-1"
    }

    tools { 
        maven 'Maven_3.8.4'  
    }

    stages {
        stage('Compile and Run Sonar Analysis') {
            steps { 
                sh 'mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=iphone_iphone -Dsonar.organization=iphone -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=34b116ff9535ddaf409e48acc7f1245327112470'
            }
        }

        stage('Build Docker Image') { 
            steps { 
                script {
                    // Tag the image with the full ECR path directly
                    dockerImage = docker.build("${ECR_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    // Log in to ECR root URL and push both build number and latest tags
                    docker.withRegistry("https://${ECR_REGISTRY}", "ecr:${AWS_REGION}:aws-credentials") {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up the local image after push to prevent filling up EC2 disk space
            sh "docker rmi ${ECR_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} ${ECR_REGISTRY}/${IMAGE_NAME}:latest || true"
        }
    }
}
