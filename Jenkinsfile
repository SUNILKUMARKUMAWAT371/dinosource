pipeline {
    agent any
    environment {
        ENVIRONMENT = getEnvironmentName(env.BRANCH_NAME)

        // Global variables
        GCP_PROJECT_ID_CRED_ID = "finance_${ENVIRONMENT}_gcp_project_id"
        GCP_REGION_CRED_ID = "finance_${ENVIRONMENT}_gcp_region"
        DOCKER_REGISTRY_CRED_ID = "finance_${ENVIRONMENT}_docker_registry_name"

        // Admin Variables
        ADMIN_DOCKER_NAME_CRED_ID = "finance_${ENVIRONMENT}_admin_docker_name"
        ADMIN_CLOUDRUN_SERVICE_CRED_ID = "finance_${ENVIRONMENT}_admin_cloudrun_service_name"
        ADMIN_SONARQUBE_PROJECT_CRED_ID = "finance_${ENVIRONMENT}_admin_sonarqube_project"
    }
    
    stages {
        stage('Retrieve Credentials') {
            steps {
                script {
                    echo "Triggered from branch: ${env.BRANCH_NAME}"
                    echo "Using environment: ${ENVIRONMENT}"

                    withCredentials([
                        // string(credentialsId: ADMIN_DOCKER_NAME_CRED_ID, variable: 'ADMIN_DOCKER_NAME'),
                        usernamePassword(credentialsId: GCP_PROJECT_ID_CRED_ID, usernameVariable: 'GCP_PROJECT', passwordVariable: 'GCP_PROJECT_ID'),
                        usernamePassword(credentialsId: GCP_REGION_CRED_ID, usernameVariable: 'REGION', passwordVariable: 'GCP_REGION'),
                        usernamePassword(credentialsId: DOCKER_REGISTRY_CRED_ID, usernameVariable: 'DOCKER_REGISTRY', passwordVariable: 'DOCKER_REGISTRY_NAME'),

                        usernamePassword(credentialsId: ADMIN_DOCKER_NAME_CRED_ID, usernameVariable: 'DOCKER_IMAGE', passwordVariable: 'DOCKER_IMAGE_NAME'),
                        usernamePassword(credentialsId: ADMIN_CLOUDRUN_SERVICE_CRED_ID, usernameVariable: 'CLOUDRUN_SERVICE', passwordVariable: 'CLOUDRUN_SERVICE_NAME'),
                        usernamePassword(credentialsId: ADMIN_SONARQUBE_PROJECT_CRED_ID, usernameVariable: 'SONARQUBE', passwordVariable: 'SONARQUBE_PROJECT'),
                    ]) {
                        echo "GCP Project ID: $GCP_PROJECT_ID"
                        echo "GCP Region: $GCP_REGION"
                        echo "Docker Registry: $DOCKER_REGISTRY_NAME"
                        echo "Admin Docker Image: $DOCKER_IMAGE_NAME"
                        echo "Admin Cloud Run Service: $CLOUDRUN_SERVICE_NAME"
                        echo "SonarQube Project: $SONARQUBE_PROJECT"
                    }
                }
            }
        }
    }
}

def getEnvironmentName(branch) {
    if (branch == 'refs/remotes/origin/Staging') {
        return 'staging'
    } else if (branch == 'pre-prod') {
        return 'preprod'
    } else if (branch == 'prod') {
        return 'prod'
    } else {
        return 'staging'
    }
}