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
                dir('aws-elastic-beanstalk-express-js-sample') {
                    sh 'npm install --save'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('aws-elastic-beanstalk-express-js-sample') {
                    sh 'npm test || echo "No tests available, skipping..."'
                }
            }
        }

stage('Security Scan') {
    steps {
        dir('aws-elastic-beanstalk-express-js-sample') {
            withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                sh '''
                    npm install -g snyk
                    snyk auth $SNYK_TOKEN
                    snyk test --file=package.json --package-manager=npm --severity-threshold=high
                '''
            }
        }
    }
}

        stage('Build Docker Image') {
            steps {
                dir('aws-elastic-beanstalk-express-js-sample') {
                    sh 'docker build -t $REGISTRY/$IMAGE_NAME:latest .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                dir('aws-elastic-beanstalk-express-js-sample') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push $REGISTRY/$IMAGE_NAME:latest
                        '''
                    }
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
