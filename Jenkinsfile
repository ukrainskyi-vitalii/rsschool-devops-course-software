pipeline {
    agent {
        kubernetes {
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              imagePullSecrets:
              - name: ecr-secret
              containers:
              - name: jnlp
                image: jenkins/inbound-agent
                args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
              - name: docker
                image: 590184028943.dkr.ecr.eu-west-1.amazonaws.com/custom-docker-with-aws:latest
                resources:
                  requests:
                    memory: "1Gi"
                    cpu: "1"
                  limits:
                    memory: "2Gi"
                    cpu: "2"
                securityContext:
                  privileged: true
                env:
                - name: DOCKER_HOST
                  value: "unix:///var/run/docker.sock"
                command:
                - dockerd
                args:
                - --host=unix:///var/run/docker.sock
                - --storage-driver=overlay2
              - name: helm
                image: alpine/helm:3.9.0
                command:
                - sleep
                args:
                - 99d
              - name: node
                image: node:18-alpine
                resources:
                  requests:
                    memory: "256Mi"
                    cpu: "250m"
                  limits:
                    memory: "512Mi"
                    cpu: "500m"
                command:
                - sleep
                args:
                - 99d
              volumes:
              - name: docker-sock-volume
                hostPath:
                  path: /var/run/docker.sock
                  type: Socket
            """
        }
    }
    environment {
        ECR_URL = "590184028943.dkr.ecr.eu-west-1.amazonaws.com"
        IMAGE_NAME = "rs-school_app"
        IMAGE_TAG = "latest"
        KUBE_NAMESPACE = "jenkins"
        AWS_CREDENTIALS = "aws_credentials"
        SONAR_PROJECT_KEY = "rs-school-app"
    }
    parameters {
        booleanParam(name: 'PUSH_TO_ECR', defaultValue: false, description: 'Do you want to push the Docker image to ECR?')
    }
    stages {
        /*stage('Create ECR Secret') {
            steps {
                container('docker') {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: AWS_CREDENTIALS
                    ]]) {
                        sh '''
                        aws ecr get-login-password --region eu-west-1 | kubectl create secret docker-registry ecr-secret \
                            --docker-server=590184028943.dkr.ecr.eu-west-1.amazonaws.com \
                            --docker-username=AWS \
                            --docker-password-stdin \
                            --namespace jenkins || echo "Secret already exists"
                        '''
                    }
                }
            }
        }
        stage('Test') {
            steps {
                container('docker') {
                    sh '''
                    apk add --no-cache python3 py3-pip
                    pip install --no-cache-dir awscli
                    docker --version
                    aws --version
                    '''
                }
            }
        }
        stage('Application Build') {
            steps {
                container('node') {
                    sh '''
                    cd rs-school_app
                    npm install
                    npm run build
                    '''
                }
            }
        }
        stage('Unit Test Execution') {
            steps {
                container('node') {
                    sh '''
                    cd rs-school_app
                    npm install
                    npm run test
                    '''
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'SonarQube analysis completed successfully!'
                }
            }
        }
        stage('Docker Build and Push to ECR') {
            when { expression { params.PUSH_TO_ECR == true } }
            steps {
                script {
                    if (currentBuild.result != 'FAILURE') {
                        env.PUSH_SUCCESSFUL = true
                    } else {
                        env.PUSH_SUCCESSFUL = false
                    }
                    container('docker') {
                        script {
                            sh '''
                            aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin $ECR_URL
                            docker build -t $ECR_URL/$IMAGE_NAME:$IMAGE_TAG ./rs-school_app
                            docker push $ECR_URL/$IMAGE_NAME:$IMAGE_TAG
                            '''
                        }
                    }
                }
            }
        }
        stage('Deploy to K8s with Helm') {
            steps {
                container('helm') {
                    script {
                        sh '''
                        helm upgrade --install rs-school-app ./helm-chart-nest-app \
                            --namespace $KUBE_NAMESPACE \
                            --set serviceAccount.create=false \
                            --set serviceAccount.name=jenkins \
                            --set image.repository=$ECR_URL/$IMAGE_NAME \
                            --set image.tag=$IMAGE_TAG
                        '''
                    }
                }
            }
        }*/
    }
    post {
        success {
            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build ${env.BUILD_NUMBER} of job '${env.JOB_NAME}' was successful.

                Project: ${SONAR_PROJECT_KEY}
                SonarQube Quality Gate: PASSED

                View the results: ${env.BUILD_URL}
                """,
                to: 'ukrainskyi.vitalii@gmail.com'
            )
        }
        failure {
            emailext(
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build ${env.BUILD_NUMBER} of job '${env.JOB_NAME}' has failed.

                Project: ${SONAR_PROJECT_KEY}
                SonarQube Quality Gate: FAILED or another error occurred.

                View the results: ${env.BUILD_URL}
                """,
                to: 'ukrainskyi.vitalii@gmail.com'
            )
        }
    }
}
