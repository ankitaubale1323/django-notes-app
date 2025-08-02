@Library('shared-lib') _

pipeline {
    agent { label 'web' }

    environment {
        IMAGE_NAME = 'notes-app:latest'
        CONTAINER_NAME = 'notes-app-container'
        DB_NAME = 'note-app-db'
        DB_USER = 'root'
        DB_PASSWORD = 'root'
        DB_HOST = 'db'          // Use service name for internal Docker networking
        DB_PORT = '3306'
    }

    stages {
         stage('Display') {

            steps {
            echo" script is running"
              }
            }
        
        
        stage('Clone Repository') {

            steps {
              script{
                     clone ("https://github.com/ankitaubale1323/django-notes-app.git", "main")
              }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image for Django app...'
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo '📦 Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'docker_hub_cred', usernameVariable: 'DockerHubUser', passwordVariable: 'DockerHubPass')]) {
                    sh 'docker login -u ${DockerHubUser} -p ${DockerHubPass}'
                    sh "docker tag ${IMAGE_NAME} ${DockerHubUser}/notes-app:latest"
                    sh "docker push ${DockerHubUser}/notes-app:latest"
                }
            }
        }

        stage('Clean Existing Containers') {
            steps {
                echo '🧹 Stopping and removing existing containers (if any)...'
                sh 'docker compose down || true'
            }
        }

        stage('Deploy using Docker Compose') {
            steps {
                echo '🚀 Deploying containers (nginx, django, mysql)...'
                sh 'docker compose up -d --build'
            }
        }

        stage('Verify Deployment') {
            steps {
                echo '🔍 Verifying running containers...'
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD deployment complete! Visit: http://<your-ec2-ip>'
        }
        failure {
            echo '❌ CI/CD pipeline failed. Check the Jenkins logs for errors.'
        }
    }
}
