pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.23.2-debug
    command:
      - /busybox/cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker

  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-secret
'''
        }
    }

    environment {
        IMAGE_NAME = "sarthak24shirbhate/enterprise-python-app"
        IMAGE_TAG = "v1"
    }

    stages {

        stage('Clone') {
            steps {
                git branch: 'main',
                url: 'https://github.com/sarthak24shirbhate/enterprise-devops-platform.git'
            }
        }

        stage('Build and Push Image') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --context `pwd` \
                      --dockerfile `pwd`/Dockerfile \
                      --destination=$IMAGE_NAME:$IMAGE_TAG \
                      --insecure \
                      --skip-tls-verify
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/enterprise-python-app \
                enterprise-python-app=$IMAGE_NAME:$IMAGE_TAG \
                -n default
                '''
            }
        }
    }
}
