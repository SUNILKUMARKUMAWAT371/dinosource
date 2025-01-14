
pipeline {
    agent { label "GCP-JENKINS-AGENT" }

    stages {
        stage('Build') {
            steps {
                sh 'ls -la'
                sh 'docker build -t dinosaur-img:v1 .'
            }
        }

        stage('Deploy') {
            steps {
                sh 'echo "successfully build"'
            }
        }
    }
}

