pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')  // Add this in Jenkins credentials
        GITHUB_CREDENTIALS = credentials('github-cred')  // GitHub PAT as username/password
        IMAGE_NAME = 'kamalfarah/hello-world'
        IMAGE_TAG = "${env.BUILD_ID}"  // Or 'latest' for testing
        REPO_URL = 'https://github.com/kamalfarah/my-cicd-test-repo.git'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build and Test') {
            steps {
                sh 'npm install'
                // Add tests if needed: sh 'npm test'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Update Manifest and Push to Git') {
            steps {
                sh """
                git config --global user.email "jenkins@example.com"
                git config --global user.name "Jenkins"
                sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml
                git add k8s/deployment.yaml
                git commit -m "Update image tag to ${IMAGE_TAG}"
                git push https://${GITHUB_CREDENTIALS_USR}:${GITHUB_CREDENTIALS_PSW}@github.com/kamalfarah/my-cicd-test-repo.git HEAD:main
                """
            }
        }
    }
}
