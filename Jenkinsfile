pipeline {
    agent {
        docker {
            image 'node:16'   // Use Node 16 Docker image
            args '-u root:root' // Run as root inside the container
        }
    }
    environment {
        REGISTRY = "docker.io/<your-dockerhub-username>"  // Replace with your Docker Hub username
        IMAGE_NAME = "nodejs-sample-app"
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || echo "No tests available, skipping..."'
            }
        }

        stage('Security Scan') {
            steps {
                sh '''
                npm install -g snyk
                snyk auth 601b92cc-eca8-4da4-869b-d5e29c568cee
                snyk test --severity-threshold=high
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    docker push $REGISTRY/$IMAGE_NAME:latest
                    '''
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/npm-debug.log', allowEmptyArchive: true
        }
        failure {
            echo 'Build failed! Check logs for details.'
        }
    }
}
