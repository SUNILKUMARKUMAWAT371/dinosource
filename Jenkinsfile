pipeline {
    agent any
    environment {
        ENVIRONMENT = getEnvironmentName(env.BRANCH_NAME)
        GCP_PROJECT_ID_CRED_ID = "finance_${ENVIRONMENT}_gcp_project_id"
        GCP_REGION_CRED_ID = "finance_${ENVIRONMENT}_gcp_region"
        DOCKER_REGISTRY_CRED_ID = "finance_${ENVIRONMENT}_docker_registry_name"

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
                        string(credentialsId: GCP_PROJECT_ID_CRED_ID, variable: 'GCP_PROJECT_ID'),
                        string(credentialsId: GCP_REGION_CRED_ID, variable: 'GCP_REGION'),
                        string(credentialsId: DOCKER_REGISTRY_CRED_ID, variable: 'DOCKER_REGISTRY'),
                        
                        string(credentialsId: ADMIN_DOCKER_NAME_CRED_ID, variable: 'ADMIN_DOCKER_NAME'),
                        string(credentialsId: ADMIN_CLOUDRUN_SERVICE_CRED_ID, variable: 'ADMIN_CLOUDRUN_SERVICE'),
                        string(credentialsId: ADMIN_SONARQUBE_PROJECT_CRED_ID, variable: 'ADMIN_SONARQUBE_PROJECT')
                    ]) {
                        echo "GCP Project ID: $GCP_PROJECT_ID"
                        echo "GCP Region: $GCP_REGION"
                        echo "Docker Registry: $DOCKER_REGISTRY"
                        echo "Admin Docker Image: $ADMIN_DOCKER_NAME"
                        echo "Admin Cloud Run Service: $ADMIN_CLOUDRUN_SERVICE"
                        echo "SonarQube Project: $ADMIN_SONARQUBE_PROJECT"
                    }
                }
            }
        }
    }
}

def getEnvironmentName(branch) {
    if (branch == 'Staging') {
        return 'Staging'
    } else if (branch == 'pre-prod') {
        return 'preprod'
    } else if (branch == 'prod') {
        return 'prod'
    } else {
        return 'Staging'
    }
}