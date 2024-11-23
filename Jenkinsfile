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
                image: docker
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
              - name: node
                image: node:18-alpine
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
        stage('Test') {
          steps {
            container('docker') {
              sh 'docker --version'
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
    }
}
