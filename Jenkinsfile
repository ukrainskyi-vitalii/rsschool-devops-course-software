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
    }
    stages {
        stage('Test') {
            steps {
                container('docker') {
                    sh 'docker --version'
                    sh 'aws --version'
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
        /*stage('Approve Docker Push') {
            steps {
                input message: 'Do you want to push the Docker image to ECR?'
            }
        }*/
        stage('Docker Build and Push to ECR') {
            steps {
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
        stage('Deploy to K8s with Helm') {
            steps {
                container('helm') {
                    script {
                        sh '''
                        helm upgrade --install rs-school-app ./helm-chart-nest-app \
                            --namespace $KUBE_NAMESPACE \
                            --set image.repository=$ECR_URL/$IMAGE_NAME \
                            --set image.tag=$IMAGE_TAG
                        '''
                    }
                }
            }
        }
    }
}
