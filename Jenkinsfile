pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "vishravi1975/springmvc-cicd-argo"
        GITOPS_REPO = "https://github.com/vishravi2016/springmvc-argocd-cicd-demo.git"
        GITOPS_BRANCH = "main"
    }

    stages {

        stage('Checkout Application Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vishravi2016/springmvc-argocd-cicd-demo.git'
            }
        }

        stage('Build Spring Boot Application') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Update GitOps Repository') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh """
                    rm -rf springmvc-argocd-cicd-demo
                    git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/vishravi2016/springmvc-argocd-cicd-demo.git

                    cd springmvc-argocd-cicd-demo

                    sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|g' k8s/deployment.yaml

                    git config user.email "jenkins@example.com"
                    git config user.name "jenkins"

                    git add k8s/deployment.yaml
                    git commit -m "Update image to ${DOCKER_IMAGE}:${BUILD_NUMBER}" || echo "No changes to commit"
                    git push origin ${GITOPS_BRANCH}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'CI completed. ArgoCD will now deploy using GitOps.'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output.'
        }
    }
}