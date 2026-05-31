pipeline {
agent any

```
environment {
    AWS_REGION = "ap-south-1"
    CLUSTER_NAME = "bluegreen-eks"
    NAMESPACE = "production"

    DOCKER_IMAGE = "vikash3117/sample-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
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

    stage('Push Docker Image') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    docker logout
                """
            }
        }
    }

    stage('Deploy To EKS') {
        steps {
            withCredentials([
                [
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS-Creds'
                ]
            ]) {

                sh """
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${CLUSTER_NAME}

                    echo "===== VERIFY NODES ====="
                    kubectl get nodes

                    NODE_COUNT=\$(kubectl get nodes --no-headers | wc -l)

                    if [ "\$NODE_COUNT" -lt 1 ]; then
                        echo "No worker nodes available"
                        exit 1
                    fi

                    echo "===== CREATE NAMESPACE ====="
                    kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -

                    echo "===== DEPLOY GREEN VERSION ====="

                    cat <<EOF | kubectl apply -f -
```

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

* port: 80
  targetPort: 8080
  EOF

  ```
                  echo "===== VALIDATE DEPLOYMENT ====="
                  kubectl rollout status deployment/app-green -n ${NAMESPACE}

                  kubectl get pods -n ${NAMESPACE}
                  kubectl get svc -n ${NAMESPACE}

                  echo "===== DELETE BLUE DEPLOYMENT ====="
                  kubectl delete deployment app-blue -n ${NAMESPACE} --ignore-not-found=true
              """
          }
      }
  }
  ```

  }

  post {
  success {
  echo "Blue-Green deployment completed successfully"
  }

  ```
  failure {
      echo "Deployment failed"
  }
  ```

  }
  }
