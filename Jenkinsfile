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
      - name: workspace-volume
        mountPath: /workspace

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - cat
    tty: true

  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-secret

    - name: workspace-volume
      emptyDir: {}
'''
        }
    }

    environment {
        IMAGE_NAME = "sarthak24shirbhate/enterprise-python-app"
        IMAGE_TAG = "v1"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                url: 'https://github.com/sarthak24shirbhate/enterprise-devops-platform.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                pwd
                ls -la
                find . -name Dockerfile
                '''
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --context=$(pwd)/app \
                      --dockerfile=$(pwd)/app/Dockerfile \
                      --destination=$IMAGE_NAME:$IMAGE_TAG \
                      --insecure \
                      --skip-tls-verify
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl set image deployment/enterprise-python-app \
                    enterprise-python-app=$IMAGE_NAME:$IMAGE_TAG \
                    -n default

                    kubectl rollout status deployment/enterprise-python-app -n default
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl get pods -n default
                    kubectl get svc -n default
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
