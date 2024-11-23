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
              - name: custom-docker-with-aws
                image: 590184028943.dkr.ecr.eu-west-1.amazonaws.com/custom-docker-with-aws
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
            container('custom-docker-with-aws') {
              sh 'docker --version'
              sh 'aws --version'
            }
          }
        }
    }
}
