pipeline {
    agent any

    environment {
        APP_NAME = "springboot-demo"
        HARBOR_HOST = "harbor.local.com"
        HARBOR_PROJECT = "demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${HARBOR_HOST}/${HARBOR_PROJECT}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/Conner735/springboot-demo.git'
            }
        }

        stage('Build Jar') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t ${IMAGE_NAME} .
                '''
            }
        }

        stage('Push To Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-robot', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                    sh '''
                      echo "${HARBOR_PASS}" | docker login ${HARBOR_HOST} -u "${HARBOR_USER}" --password-stdin
                      docker push ${IMAGE_NAME}
                      docker logout ${HARBOR_HOST}
                    '''
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                      export KUBECONFIG=${KUBECONFIG_FILE}

                      sed "s#REPLACE_IMAGE#${IMAGE_NAME}#g" k8s/deployment.yaml > k8s/deployment-rendered.yaml

                      kubectl apply -f k8s/deployment-rendered.yaml
                      kubectl apply -f k8s/service.yaml

                      kubectl rollout status deployment/${APP_NAME} -n default
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline success: ${IMAGE_NAME}"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
