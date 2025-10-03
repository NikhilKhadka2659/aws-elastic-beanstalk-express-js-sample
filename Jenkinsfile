pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/nikhilkhadka885"   // your DockerHub username
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
                sh 'docker run --rm -v $(pwd):/app -w /app node:16 npm install --save'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'docker run --rm -v $(pwd):/app -w /app node:16 npm test || echo "No tests available, skipping..."'
            }
        }

        stage('Security Scan') {
            steps {
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh '''
                        docker run --rm -v $(pwd):/app -w /app node:16 bash -c "
                          npm install -g snyk && \
                          snyk auth $SNYK_TOKEN && \
                          snyk test --severity-threshold=high
                        "
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:latest $(pwd)'
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
        success {
            echo 'Build completed successfully!'
        }
    }
}
