pipeline {
    agent { label "Jenkins-Agent" }

    environment {
        APP_NAME = "register-app-pipeline"
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git(
                    branch: 'main',
                    credentialsId: 'github',
                    url: 'https://github.com/manimozhyan/gitops-register-app'
                )
            }
        }

        stage("Update the Deployment Tags") {
            steps {
                sh """
                    echo "IMAGE_TAG = ${IMAGE_TAG}"

                    cat deployment.yaml

                    sed -i 's|${APP_NAME}:.*|${APP_NAME}:${IMAGE_TAG}|g' deployment.yaml

                    cat deployment.yaml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                    git config --global user.name "manimozhyan"
                    git config --global user.email "manimozhyan.v@gmail.com"

                    git add deployment.yaml

                    if git diff --cached --quiet; then
                        echo "No changes to commit"
                    else
                        git commit -m "Updated Deployment Manifest"
                        git push https://github.com/manimozhyan/gitops-register-app main
                    fi
                """
            }
        }
    }
}
