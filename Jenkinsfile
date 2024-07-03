pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockercred'
        DOCKERHUB_REPO = 'lokesh220/pipeline'
        KIND_CLUSTER_NAME = 'new'
        ARGOCD_CLI_PATH = 'C:/Users/lokes/argocd.exe'
        REPO_URL = 'https://github.com/lokesh2201013/DevopsProject'
        KUBECONFIG_PATH = 'kubeconfig'
        DEST_SERVER = 'https://kubernetes.default.svc'
        DEST_NAMESPACE = 'default'
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

        stage('Cleanup Docker') {
            steps {
                script {
                    // Clean up Docker containers
                    bat 'docker stop pipeline'
                    bat 'docker rm pipeline'
                }
            }
        }

        stage('Check Kind Cluster') {
            steps {
                script {
                    // Check if Kind cluster "new" exists
                    def result = bat(script: 'kind get clusters | findstr "new" > nul && echo exists || echo not exists', returnStatus: true)
                    echo "Cluster status: ${result}"
                    if (result != 0) {
                        echo "Kind cluster 'new' does not exist. Creating..."
                        bat 'kind create cluster --name new'
                    } else {
                        echo "Kind cluster 'new' exists."
                    }
                }
            }
        }

        stage('Argo CD Integration') {
            steps {
                script {
                    // Login to Argo CD
                    bat "${ARGOCD_CLI_PATH} login cd.argoproj.io --core"

                    // Create or sync Argo CD application
                    bat "${ARGOCD_CLI_PATH} app create pipeline --repo ${REPO_URL} --path ${KUBECONFIG_PATH} --dest-server ${DEST_SERVER} --dest-namespace ${DEST_NAMESPACE}"
                    bat "${ARGOCD_CLI_PATH} app sync pipeline"
                }
            }
        }

        stage('Final Cleanup and Notification') {
            steps {
                script {
                    // Additional cleanup or notifications
                    echo 'Performing final cleanup and notification steps...'
                    
                    // Example: Send email notification
                    mail to: 'lokeshchoraria60369@gmail.com', subject: 'Pipeline Build Status', body: 'The pipeline build has completed successfully.'

                    // Example: Any other cleanup steps if needed
                }
            }
        }
    }
}
