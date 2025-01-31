pipeline {
    agent { label "GCP-JENKINS-AGENT" }

    // parameters {
    //     booleanParam(name: 'SONARQUBE_CODE_QUALITY', defaultValue: true, description: 'SonarQube code Analysis')
    //     booleanParam(name: 'TRIVY_SCANING', defaultValue: true, description: 'Trivy Docker Image Scanning')
    //     booleanParam(name: 'FILESYSTEM_SCANNING', defaultValue: true, description: 'File System Scanning')
    //     booleanParam(name: 'CLOUDRUN_DEPLOYMENT', defaultValue: true, description: 'CloudRun Deployment')
    //     choice(name: 'BRANCH_NAME', choices: ['dev', 'main', 'staging'], description: 'Select Branch to Build')
    // }

    environment {
        SONAR_HOME = tool 'Sonar'
        PROJECT_ID = 'adept-protocol-441916-r0'
        REGION = 'asia-south1'
        REPO_NAME = 'dinosaur'
        IMAGE_NAME = 'dinosaur'
        IMAGE_TAG = '0.0.10'
        SERVICE_NAME = 'dinosaur-svc'
    }

    stages {
        stage('Load Parameters from .env dependencies') {
            steps {
                script {
                    def envFile = readFile('.env')
                    def envVars = envFile.split('\n').collect { line ->
                        def parts = line.split('=')
                        if (parts.size() == 2) {
                            def key = parts[0].trim()
                            def value = parts[1].trim()
                            if (key && value) {
                                env[key] = value
                            }
                        }
                    }
                    echo "Loaded environment variables from .env file"
                }
            }
        }

        stage('Write Secret to File') {
            steps {
                script {
                    // Access the secret text using withCredentials
                    withCredentials([string(credentialsId: 'dinosaur.env.dev', variable: 'MY_SECRET')]) {
                        // Write the secret to the env.dev file
                        sh '''
                        echo "${MY_SECRET}" > env.dev
                        ls -la
                        cat env.dev
                        git branch -a
                        echo $GIT_BRANCH
                        '''
                    }
                }
            }
        }

        // stage('Google Cloud Auth') {
        //     steps {
        //         withCredentials([file(credentialsId: 'dinosaur-sa-key', variable: 'GCP_Service_Account')]) {
        //             sh """
        //                 gcloud auth activate-service-account --key-file=${GCP_Service_Account}
        //                 gcloud config set project $PROJECT_ID
        //                 gcloud auth configure-docker $REGION-docker.pkg.dev
        //             """
        //         }
        //     }
        // }

        stage('SonarQube Code Quality Analysis') {
            when {
                //expression { params.SONARQUBE_CODE_QUALITY == true }
                expression { env.SONARQUBE_CODE_QUALITY.toBoolean() }
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
                //expression { params.FILESYSTEM_SCANNING == true }
                expression { env.FILESYSTEM_SCANNING.toBoolean() }
            }
            steps {
                echo 'Performing file system scan...'
                sh 'ls -la'
            }
        }

        // stage('Build Docker Image') {
        //     steps {
        //         sh 'docker build -t $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG .'
        //     }
        // }

        stage('Trivy Scan') {
            when {
                //expression { params.TRIVY_SCANING == true }
                expression { env.TRIVY_SCANING.toBoolean() }
            }
            steps {
                echo 'Running Trivy vulnerability scan...'
                sh 'trivy fs . --exit-code 0'
            }
        }

        // stage('Push to Artifact Registry') {
        //     steps {
        //         sh 'docker push $REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG'
        //     }
        // }

        stage('Deploy to Cloud Run') {
            when {
                //expression { params.CLOUDRUN_DEPLOYMENT == true }
                expression { env.CLOUDRUN_DEPLOYMENT.toBoolean() }
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
