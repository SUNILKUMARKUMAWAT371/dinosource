pipeline {
    agent { label "GCP-JENKINS-AGENT" }

    parameters {
        booleanParam(name: 'RUN_SONAR', defaultValue: false, description: 'Run SonarQube Analysis')
        booleanParam(name: 'RUN_TRIVY', defaultValue: false, description: 'Run Trivy Scan')
        booleanParam(name: 'RUN_FILESYSTEM_SCAN', defaultValue: false, description: 'Run File System Scan')
        booleanParam(name: 'RUN_DOCKER_DEPLOY', defaultValue: true, description: 'Run Docker Compose Deploy')
        choice(name: 'BRANCH_NAME', choices: ['dev', 'main', 'staging'], description: 'Select Branch to Build')
    }

    environment {
        SONAR_HOME = tool 'Sonar'
        PROJECT_ID = 'adept-protocol-441916-r0'
        REGION = 'asia-south1'
        REPO_NAME = 'dinosaur'
        IMAGE_NAME = 'dinosaur'
        IMAGE_TAG = '0.0.8'
        SERVICE_NAME = 'dinosaur-svc'
    }

    stages {
        
        stage('Google Cloud Auth') {
            steps {
                withCredentials([file(credentialsId: 'dinosaur-sa-key', variable: 'GCP_Service_Account')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=${GCP_Service_Account}
                        gcloud config set project $PROJECT_ID
                        gcloud auth configure-docker $REGION-docker.pkg.dev
                    """
                }
            }
        }

        stage('SonarQube Code Quality Analysis') {
            when {
                expression { params.RUN_SONAR }
            }
            steps {
                echo 'Running SonarQube Analysis...'
                withSonarQubeEnv('Sonar') {
                    sh '$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=dinosaur -Dsonar.projectKey=dinosaur'
                }
            }
        }

        stage('File System Scan') {
            when {
                expression { params.RUN_FILESYSTEM_SCAN }
            }
            steps {
                echo 'Performing file system scan...'
                sh 'ls -la'
            }
        }

        stage('Trivy Scan') {
            when {
                expression { params.RUN_TRIVY }
            }
            steps {
                echo 'Running Trivy vulnerability scan...'
                sh 'trivy fs . --exit-code 1'
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
            when {
                expression { params.RUN_DOCKER_DEPLOY }
            }
            steps {
                sh """
                    gcloud run deploy $SERVICE_NAME \
                        --image=$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG \
                        --region=$REGION \
                        --platform=managed \
                        --allow-unauthenticated
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
            emailext body: 'Pipeline executed successfully!',
                     subject: 'Pipeline Success',
                     to: 'sunil.kumawat@thewitslab.com'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
            emailext body: 'Pipeline execution failed. Please check Jenkins logs.',
                     subject: 'Pipeline Failed',
                     to: 'sunil.kumawat@thewitslab.com'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}



// pipeline {
//     agent { label "GCP-JENKINS-AGENT" }

//     parameters {
//         booleanParam(name: 'RUN_SONAR', defaultValue: false, description: 'Run SonarQube Analysis')
//         booleanParam(name: 'RUN_TRIVY', defaultValue: false, description: 'Run Trivy Scan')
//         booleanParam(name: 'RUN_FILESYSTEM_SCAN', defaultValue: false, description: 'Run File System Scan')
//         booleanParam(name: 'RUN_DOCKER_DEPLOY', defaultValue: true, description: 'Run Docker Compose Deploy')
//         choice(name: 'BRANCH_NAME', choices: ['dev', 'main', 'staging'], description: 'Select Branch to Build')
//     }

//     environment {
//         SONAR_HOME = tool 'Sonar'
//         PROJECT_ID = 'adept-protocol-441916-r0' // gcp project id
//         REGION = 'asia-south1'                  // region name
//         REPO_NAME = 'dinosaur'                  // docker registry name
//         IMAGE_NAME = 'dinosaur'                 // docker image name
//         IMAGE_TAG='0.0.8'                       // docker image tag
//         SERVICE_NAME = 'dinosaur-svc'           // cloud run service id
//     }

//     stages {
//         stage('Google Cloud Auth') {
//             steps {
//                 withCredentials([file(credentialsId: 'dinosaur-sa-key', variable: 'GCP_Service_Account')]) {
//                     sh("gcloud auth activate-service-account --key-file=${GCP_Service_Account}")
//                     sh 'gcloud config set project $PROJECT_ID'
//                 }
//             }
//         }
        
//         stage('SonarQube Code Quality Analysis') {
//             when {
//                 expression { params.RUN_SONAR == true }
//             }
//             steps {
//                 echo 'Doing Sonar Analysis...'
//                 withSonarQubeEnv('Sonar') {
//                     sh '$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=dinosaur -Dsonar.projectKey=dinosaur'
//                 }
//             }
//         }

//         stage('File System Scan') {
//             when {
//                 expression { params.RUN_FILESYSTEM_SCAN == true }
//             }
//             steps {
//                 echo 'Scanning the file system...'
//                 sh 'ls -la'
//             }
//         }

//         stage('Trivy Scan') {
//             when {
//                 expression { params.RUN_TRIVY == true }
//             }
//             steps {
//                 echo 'Scanning Docker image for vulnerabilities...'
//                 sh "trivy fs . || exit 1"
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 sh 'docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG .'
//             }
//         }

//         stage('Push to Artifact Registry') {
//             steps {
//                 sh 'docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG'
//             }
//         }

//         stage('Deploy to Cloud Run') {
//             steps {
//                 sh '''
//                     gcloud run deploy $SERVICE_NAME \
//                         --image=$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG \
//                         --region=$REGION \
//                         --platform=managed \
//                         --allow-unauthenticated
//                 '''
//             }
//         }

//         post {
//             always {
//                 cleanWs()
//             }
//        }
//     }

//     post {
//         always {
//             echo 'Cleaning up workspace...'
//             cleanWs()
//         }
//         success {
//             echo 'Pipeline completed successfully!'
//             emailext body: 'Pipeline executed successfully!',
//                     subject: 'Pipeline Success',
//                     to: 'sunil.kumawat@thewitslab.com'
//         }
//         failure {
//             echo 'Pipeline failed. Please check the logs.'
//             emailext body: 'Pipeline execution failed. Please check Jenkins logs.',
//                     subject: 'Pipeline Failed',
//                     to: 'sunil.kumawat@thewitslab.com'
//         }
//     }
// }