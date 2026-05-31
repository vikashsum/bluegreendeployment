pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        CLUSTER_NAME = "bluegreen-eks"
        NAMESPACE = "production"

        DOCKER_IMAGE = "vikash3117/sample-app"
        IMAGE_TAG = "${BUILD_NUMBER}"

        KUBECONFIG = "${env.WORKSPACE}/kubeconfig"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                """
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    docker logout
                    """
                }
            }
        }

        stage('Configure AWS & Kubeconfig') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh """
                    aws eks update-kubeconfig \
                        --region ${AWS_REGION} \
                        --name ${CLUSTER_NAME}

                    kubectl get nodes
                    """
                }
            }
        }

        stage('Deploy Green Version') {
            steps {
                sh """
cat <<EOF | kubectl apply -f -

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
  namespace: ${NAMESPACE}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sample-app
      version: green
  template:
    metadata:
      labels:
        app: sample-app
        version: green
    spec:
      containers:
      - name: sample-app
        image: ${DOCKER_IMAGE}:${IMAGE_TAG}
        ports:
        - containerPort: 8080

        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10

---
apiVersion: v1
kind: Service
metadata:
  name: sample-service
  namespace: ${NAMESPACE}
spec:
  selector:
    app: sample-app
    version: green
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer

EOF
                """
            }
        }

        stage('Switch Traffic to Green (Blue-Green)') {
            steps {
                sh """
                echo "Switching traffic to GREEN..."

                kubectl patch service sample-service -n ${NAMESPACE} \
                -p '{"spec":{"selector":{"version":"green"}}}'
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                kubectl rollout status deployment/app-green -n ${NAMESPACE}
                kubectl get pods -n ${NAMESPACE}
                kubectl get svc -n ${NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Blue-Green deployment SUCCESS"
        }

        failure {
            echo "❌ Pipeline FAILED"
        }
    }
}
