pipeline {
  agent any

  environment {
    REGISTRY = "docker.io"
    DOCKERHUB_USER = "baghel1997"
    IMAGE_NAME = "argocd-demo"
    GITOPS_REPO = "git@github.com:devendra-singh2000/argocd-helm-repo.git"
    GITOPS_APP_PATH = "charts/argocd-demo"
    // Jenkins credentials IDs you will create
    DOCKERHUB_CRED = "dockerhub-creds"
    GITOPS_CRED = "gitops-deploy-key"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker image') {
      steps {
        script {
          COMMIT = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
          IMAGE_TAG = "${COMMIT}"
          env.IMAGE_TAG = IMAGE_TAG
          sh """
            docker build -t ${REGISTRY}/${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
          """
        }
      }
    }

    stage('Login & Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: DOCKERHUB_CRED, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh """
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin ${REGISTRY}
            docker push ${REGISTRY}/${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
          """
        }
      }
    }

    stage('Update GitOps repo') {
      steps {
          sh """
            git clone https://github.com/devendra-singh2000/argocd-helm-repo.git
            git branch
            cd argocd-helm-repo
            sed -i 's#image: .*#image: ${REGISTRY}/${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}#' ${GITOPS_APP_PATH}/values.yaml
            git config user.email "jenkins@example.com"
            git config user.name "Jenkins"
            git commit -am "Update image tag to ${IMAGE_TAG}" || echo "No changes to commit"
            git push origin main
          """
      }
    }
  }
}
