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
                    credentialsId: 'githubPAT',
                    url: 'https://github.com/manimozhyan/gitops-register-app'
                )
            }
        }

        stage('Update Deployment Image') {
            steps {
                sh '''
                    echo "========================================="
                    echo "IMAGE TAG RECEIVED FROM CI"
                    echo "IMAGE_TAG = ${IMAGE_TAG}"
                    echo "========================================="

                    echo "Before update:"
                    cat deployment.yaml

                    if [ -z "${IMAGE_TAG}" ]; then
                        echo "ERROR: IMAGE_TAG is empty"
                        exit 1
                    fi

                    sed -i \
                    "s|${APP_NAME}:.*|${APP_NAME}:${IMAGE_TAG}|g" \
                    deployment.yaml

                    echo "After update:"
                    cat deployment.yaml
                '''
            }
        }

        stage('Validate Deployment YAML') {
            steps {
                sh '''
                    echo "Validating deployment.yaml..."

                    if grep -q "image: ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}" deployment.yaml; then
                        echo "Image tag updated successfully."
                    else
                        echo "ERROR: Image tag was not updated correctly."
                        exit 1
                    fi
                '''
            }
        }

        stage('Push Deployment File to Git') {
            steps {
                sh '''
                    git config --global user.name "manimozhyan"
                    git config --global user.email "manimozhyan.v@gmail.com"

                    git add deployment.yaml

                    if git diff --cached --quiet; then
                        echo "========================================="
                        echo "No changes to commit."
                        echo "Deployment already contains IMAGE_TAG=${IMAGE_TAG}"
                        echo "========================================="
                    else
                        git commit -m "Update image to ${IMAGE_TAG}"

                        git push origin main
                    fi
                '''
            }
        }
    }

    post {

        success {
            echo '''
            =========================================
            CD PIPELINE SUCCESS
            GitOps repository updated successfully.
            Argo CD should detect the new commit.
            =========================================
            '''
        }

        failure {
            echo '''
            =========================================
            CD PIPELINE FAILED
            Check the stage above for the error.
            =========================================
            '''
        }
    }
}
