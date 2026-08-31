pipeline {

    agent {
        label 'Jenkins-Agent'
    }

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: '',
            description: 'Docker image tag received from CI pipeline'
        )
    }

    environment {
        APP_NAME = "register-app-pipeline"
        DOCKER_USER = "manimozhyan"
    }

    stages {

        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/manimozhyan/gitops-register-app'
                )
            }
        }

        stage('Check IMAGE_TAG') {
            steps {
                sh '''
                    echo "========================================="
                    echo "IMAGE TAG RECEIVED FROM CI"
                    echo "IMAGE_TAG = ${IMAGE_TAG}"
                    echo "========================================="

                    if [ -z "${IMAGE_TAG}" ]; then
                        echo "ERROR: IMAGE_TAG is empty!"
                        exit 1
                    fi
                '''
            }
        }

        stage('Update Deployment Image') {
            steps {
                sh '''
                    echo "========================================="
                    echo "Before Update"
                    echo "========================================="

                    cat deployment.yaml

                    echo ""
                    echo "Updating Docker image..."

                    sed -i \
                    "s|image: ${DOCKER_USER}/${APP_NAME}:.*|image: ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}|g" \
                    deployment.yaml

                    echo ""
                    echo "========================================="
                    echo "After Update"
                    echo "========================================="

                    cat deployment.yaml
                '''
            }
        }

        stage('Validate Deployment YAML') {
            steps {
                sh '''
                    echo "========================================="
                    echo "Validating deployment.yaml"
                    echo "========================================="

                    EXPECTED_IMAGE="${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"

                    echo "Expected image:"
                    echo "${EXPECTED_IMAGE}"

                    if grep -q "image: ${EXPECTED_IMAGE}" deployment.yaml; then

                        echo ""
                        echo "Image updated successfully!"

                    else

                        echo ""
                        echo "ERROR: Image was NOT updated correctly!"
                        echo ""
                        cat deployment.yaml
                        exit 1

                    fi
                '''
            }
        }

        stage('Push Deployment File to Git') {
            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {

                        sh '''
                            echo "========================================="
                            echo "Configuring Git"
                            echo "========================================="

                            git config user.name "manimozhyan"
                            git config user.email "manimozhyan.v@gmail.com"

                            git add deployment.yaml

                            if git diff --cached --quiet; then

                                echo ""
                                echo "No changes to commit."
                                echo "Deployment already contains:"
                                echo "${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"

                            else

                                echo ""
                                echo "Committing deployment.yaml..."

                                git commit \
                                    -m "Update image to ${IMAGE_TAG}"

                                echo ""
                                echo "Pushing changes to GitHub..."

                                git push \
                                    "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/manimozhyan/gitops-register-app.git" \
                                    HEAD:main

                                echo ""
                                echo "========================================="
                                echo "Git Push Successful"
                                echo "========================================="

                            fi
                        '''
                    }
                }
            }
        }
    }

    post {

        success {
            echo '''
            =========================================
            CD PIPELINE SUCCESS
            =========================================

            GitOps repository updated successfully.

            Argo CD should detect the new commit
            and deploy the new Docker image to EKS.

            =========================================
            '''
        }

        failure {
            echo '''
            =========================================
            CD PIPELINE FAILED
            =========================================

            Check the failed stage above.

            =========================================
            '''
        }
    }
}
