pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    environment {
        IMAGE_NAME = "aditya4304/multibranch-flask-app"
        GIT_USER   = "Mikey4304"
        GIT_EMAIL  = "adityapathania0319@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh """
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Update K8s Manifest') {
            when {
                branch 'main'
            }
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github-creds',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_TOKEN'
                        )
                    ]) {
                        sh """
                            set -e

                            git config user.name "${GIT_USER}"
                            git config user.email "${GIT_EMAIL}"

                            git fetch origin
                            git checkout main
                            git reset --hard origin/main

                            sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                            git add k8s/deployment.yml

                            git diff --cached --quiet || git commit -m "Updated image to ${IMAGE_TAG}"

                            git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/Mikey4304/Multi-Branch-Prod-1.git HEAD:main
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }

        failure {
            echo "Pipeline failed"
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}