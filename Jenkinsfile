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

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - /bin/sh
      - -c
      - cat
    tty: true

  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-secret
        items:
          - key: .dockerconfigjson
            path: config.json
'''
        }
    }

    environment {
        IMAGE_NAME = "sarthak24shirbhate/enterprise-python-app:v1"
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
                sh 'ls -la'
                sh 'ls -la app'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                container('kaniko') {
                    sh '''
                    echo "Checking Docker config"

                    cat /kaniko/.docker/config.json

                    /kaniko/executor \
                      --context=$(pwd)/app \
                      --dockerfile=$(pwd)/app/Dockerfile \
                      --destination=$IMAGE_NAME \
                      --skip-tls-verify
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl version --client

                    kubectl apply -f k8s/deployment.yaml

                    kubectl apply -f k8s/service.yaml

                    kubectl rollout restart deployment enterprise-python-app

                    kubectl rollout status deployment enterprise-python-app
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl get pods

                    kubectl get svc
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
    }
}
