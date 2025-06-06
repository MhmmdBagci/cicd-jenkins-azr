// This Jenkinsfile is used to build a Docker image, create a Helm chart, and push both to Azure Container Registry.
pipeline {
    agent any
    environment {
        AZURECREDENTIALS = credentials('azure-credentials')
        REPOSITORY = "tempregistrykurs1.azurecr.io"
        DOCKERIMAGE = "Muhammed Bagci_cicd_jenkins_azr" // Please change this to your desired Docker image name
    }
    stages {
        stage('Build Image') {
            steps {
                sh '''#!/bin/bash
                cd container
                echo "Changing names in welcome.html"
                echo ""
                sed -i -e "s/REPOSITORY/${REPOSITORY}/g" welcome.html && echo "REPOSITORY changed to ${REPOSITORY}"
                sed -i -e "s/DOCKERIMAGE/${DOCKERIMAGE}/g" welcome.html && echo "DOCKERIMAGE changed to ${DOCKERIMAGE}"
                sed -i -e "s/BUILD_ID/${BUILD_ID}/g" welcome.html && echo "BUILD_ID changed to ${BUILD_ID}"
                echo "Building a new container image: \$DOCKERIMAGE:$BUILD_ID"
                docker build -t ${DOCKERIMAGE}:$BUILD_ID .
                '''
            }
        }
        // We use not Helmchart at the moment, we will use flux for now
        // stage('Build Helmchart') {
        //     steps {
        //         sh '''#!/bin/bash
        //         echo "Building a new helmchart..."
        //         cd helm && mv webapp \$DOCKERIMAGE
        //         echo "Building a new helmchart: \$DOCKERIMAGE with Version: $BUILD_ID"
        //         helm package --app-version "$BUILD_ID" --version "$BUILD_ID" \$DOCKERIMAGE/ || exit 1
        //         '''
        //     }
        // }

        stage('Push Image') {
            steps {
                sh '''#!/bin/bash
                echo "Login to Azure Container Registry: \$REPOSITORY"
                docker login -u $AZURECREDENTIALS_USR -p $AZURECREDENTIALS_PSW \$REPOSITORY

                echo "Tagging the new container image: \$DOCKERIMAGE:$BUILD_ID"
                docker tag \$DOCKERIMAGE:$BUILD_ID \$REPOSITORY/\$DOCKERIMAGE:$BUILD_ID

                echo "Pushing the new container image: \$DOCKERIMAGE:\$BUILD_ID"
                docker push \$REPOSITORY/\$DOCKERIMAGE:$BUILD_ID
                '''
            }
        }
        
        // stage('Push Helmchart') {
        //     steps {
        //         sh 'echo "Helmchart will be pushed to Azure AZR..."'
        //         //TODO: Implement the Helm chart push to Azure Container Registry
        //     }
        // }
        stage('Genereate Kubernetes Manifest') {
            steps {
                sh '''#!/bin/bash
                cd k8s
                CREATOR=$(echo ${DOCKERIMAGE} | cut -f1 -d"-")
                echo "Changing names in example-flux.yaml"
                sed -e "s/example-webapp/${DOCKERIMAGE}/g" -e "s/example-ns/${CREATOR}/g" -e "s/example-repo/${REPOSITORY}/g" -e "s/example-tag/${BUILD_ID}/g" example-flux.yaml | tee flux.yaml
                echo "Changing names in example-webapp.yaml"
                sed -e "s/example-webapp/${DOCKERIMAGE}/g" -e "s/example-ns/${CREATOR}/g" -e "s/example-repo/${REPOSITORY}/g" -e "s/example-tag/${BUILD_ID}/g" example-webapp.yaml | tee webapp.yaml
                '''
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace'
            sh 'rm -rf ./*'
        }
        failure {
            echo 'Pipeline failure...'
        }
        success {
            echo 'Pipeline success...'
        }
    }
}
