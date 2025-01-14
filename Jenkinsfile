pipeline {
    agent { label "GCP-JENKINS-AGENT" }

    environment {
        PROJECT_ID = 'adept-protocol-441916-r0' // gcp project id
        REGION = 'asia-south1' // region name
        REPO_NAME = 'dinosaur' // docker registry name
        IMAGE_NAME = 'dinosaur' // docker image name
        IMAGE_TAG='0.0.5' // docker image tag
        SERVICE_NAME = 'dinosaur-svc' // cloud run service id
    }

    stages {
        stage('Google Cloud Auth') {
            steps {
                withCredentials([file(credentialsId: 'dinosaur-sa-key', variable: 'GCP_Service_Account')]) {
                    sh("gcloud auth activate-service-account --key-file=${GCP_Service_Account}")
                    sh 'gcloud config set project $PROJECT_ID'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                sh 'docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Deploy to Cloud Run') {
            steps {
                sh '''
                    gcloud run deploy $SERVICE_NAME \
                        --image=$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG \
                        --region=$REGION \
                        --platform=managed \
                        --allow-unauthenticated
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
