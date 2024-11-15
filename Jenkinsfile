pipeline {
    agent any

    environment {
        AWS_CREDENTIALS_ID = 'aws-credentials'
        ECR_REPOSITORY = '590184028943.dkr.ecr.eu-west-1.amazonaws.com/rs-school_app'
        IMAGE_TAG = "latest"
        SONARQUBE_SCANNER = 'SonarQube Scanner'
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig-credentials'
        AWS_REGION = 'eu-west-1'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ukrainskyi-vitalii/rsschool-devops-course-software/'
            }
        }

        stage('Application Build') {
            steps {
                sh "docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} ."
            }
        }

        stage('Test Docker') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Unit Test Execution') {
            steps {
                sh "docker run --rm ${ECR_REPOSITORY}:${IMAGE_TAG} npm test"
            }
        }

        stage('Security Check with SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${SONARQUBE_SCANNER} -Dsonar.projectKey=my-project -Dsonar.sources=./src"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    withCredentials([aws(credentialsId: "${AWS_CREDENTIALS_ID}")]) {
                        sh "aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin ${ECR_REPOSITORY}"
                    }
                    sh "docker push ${ECR_REPOSITORY}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                        sh "helm upgrade --install my-app ./helm-chart --set image.repository=${ECR_REPOSITORY} --set image.tag=${IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
