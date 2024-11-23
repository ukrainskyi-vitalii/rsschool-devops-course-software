pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: jnlp
                image: jenkins/inbound-agent
                args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
              - name: docker
                image: docker:24.0.2
                command:
                - sleep
                args:
                - 99d
              - name: helm
                image: alpine/helm:3.9.0
                command:
                - sleep
                args:
                - 99d
            """
        }
    }
    environment {
        ECR_URL = "590184028943.dkr.ecr.eu-west-1.amazonaws.com"
        IMAGE_NAME = "rs-school_app"
        IMAGE_TAG = "latest"
        KUBE_NAMESPACE = "default"
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Build Application') {
            steps {
                container('docker') {
                    sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./rs-school_app
                    '''
                }
            }
        }
        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                // Замініть наступну команду на команду для запуску ваших тестів
                sh 'echo "All tests passed!"'
            }
        }
        stage('Security Check with SonarQube') {
            steps {
                echo 'Running SonarQube analysis...'
                // Додайте команду для інтеграції з SonarQube
                sh 'echo "SonarQube analysis completed!"'
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                input 'Deploy Docker image to ECR?'
                container('docker') {
                    sh '''
                    aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin ${ECR_URL}
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ECR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${ECR_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes with Helm') {
            steps {
                input 'Deploy to Kubernetes?'
                container('helm') {
                    sh '''
                    helm upgrade --install ${IMAGE_NAME} ./helm-chart-nest-app \
                        --namespace ${KUBE_NAMESPACE} \
                        --set image.repository=${ECR_URL}/${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG}
                    '''
                }
            }
        }
    }
}
