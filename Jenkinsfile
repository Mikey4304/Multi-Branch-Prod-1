e
pipeline {
    agent any
 
    options {
        disableConcurrentBuilds()
        timestamps()
    }
 
    environment {
        IMAGE_NAME    = "aditya4304/multibranch-flask-app"
        GIT_USER      = "Mikey4304"
        GIT_EMAIL     = "adityapathania0319@gmail.com"
        MANIFEST_FILE = "k8s/deployment.yml"   // change to deployment.yaml if that is your file name
        REPO_PATH     = "Mikey4304/Multi-Branch-Prod-1.git"
    }
 
    stages {
 
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
 
        stage('Build and Push Docker Image') {
            when {
                allOf {
                    branch 'main'
                    // Skip builds triggered by the pipeline's own manifest commit (prevents an infinite loop)
                    not { changelog '.*\\[skip ci\\].*' }
                }
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
                            set -e
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                            docker logout
                        """
                    }
                }
            }
        }
 
        stage('Update K8s Manifest') {
            when {
                allOf {
                    branch 'main'
                    not { changelog '.*\\[skip ci\\].*' }
                }
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
 
                            git checkout main
                            git pull --rebase origin main
 
                            # Replace only the value of the image: line, keeping indentation
                            sed -i -E "s|^([[:space:]]*image:[[:space:]]*).*|\\1${IMAGE_NAME}:${IMAGE_TAG}|" ${MANIFEST_FILE}
 
                            git add ${MANIFEST_FILE}
 
                            if git diff --cached --quiet; then
                                echo "No manifest change to commit"
                            else
                                git commit -m "Updated image to ${IMAGE_TAG} [skip ci]"
                                git push https://\$GIT_USERNAME:\$GIT_TOKEN@github.com/${REPO_PATH} main
                            fi
                        """
                    }
                }
            }
        }
    }
 
    post {
        success {
            echo 'Pipeline completed successfully'
        }
 
        failure {
            echo 'Pipeline failed'
        }
 
        always {
            sh 'docker image prune -f || true'
        }
    }
