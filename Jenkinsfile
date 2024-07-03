pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockercred' // Replace with your Docker credentials ID in Jenkins
        DOCKERHUB_REPO = 'lokesh220/pipeline'
    }

    stages {
        stage('Build Image') {
            steps {
                script {
                    // Build Docker image
                    bat "docker build -t pipeline ."
                }
            }
        }

        stage('Tag Image') {
            steps {
                script {
                    // Tag the Docker image
                    bat "docker tag pipeline ${DOCKERHUB_REPO}:latest"
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        bat "docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
                    }
                    // Push the Docker image to Docker Hub
                    bat "docker push ${DOCKERHUB_REPO}:latest"
                }
            }
        }

        stage('Run Container at port 5173') {
            steps {
                script {
                    // Run Docker container
                    bat "docker run -d -p 5173:80 --name pipeline ${DOCKERHUB_REPO}:latest"
                }
            }
        }
    }

    post {
        always {
            script {
                // Check if Kind cluster "new" exists
                def result = bat(script: 'kind get clusters | findstr "new" > nul && echo exists || echo not exists', returnStatus: true)
                println "Cluster status: ${result}"

                if (result != 0) {
                    echo "Kind cluster 'new' does not exist. Creating..."
                    bat 'kind create cluster --name new'
                } else {
                    echo "Kind cluster 'new' exists."
                }

                // Clean up Docker containers
                bat 'docker stop pipeline'
                bat 'docker rm pipeline'

                // Check pods in the Kubernetes cluster
                bat 'kubectl get pods'

                // Send email notification
                mail to: 'lokeshchoraria60369@gmail.com', subject: 'Build Status', body: 'The build has completed.'
            }
        }
    }
}
