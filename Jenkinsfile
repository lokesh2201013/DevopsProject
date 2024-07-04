pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'dockercred'
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
                // Clean up Docker containers
                bat 'docker stop pipeline'
                bat 'docker rm pipeline'

                // Check if Kind cluster "new" exists
                def result = bat(script: 'kind get clusters | findstr "new" > nul && echo exists || echo not exists', returnStatus: true)
                println "Cluster status: ${result}"

                if (result != 0) {
                    echo "Kind cluster 'new' does not exist. Creating..."
                    bat 'kind create cluster --name new'
                } else {
                    echo "Kind cluster 'new' exists."
                }

                // Check if the Argo CD namespace exists
                def argoExists = bat(script: 'kubectl get namespace argocd > nul 2>&1 && echo exists || echo not exists', returnStatus: true)
                println "Argo CD namespace status: ${argoExists}"

                if (argoExists != 0) {
                    echo "Argo CD namespace does not exist. Creating..."
                    bat 'kubectl create namespace argocd'
                    bat 'kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml'
                } else {
                    echo "Argo CD namespace exists."
                }
                    def podsready=false
                    def maxtries=30
                    def retry=60
                sleep 120
                 
               
                bat 'kubectl config set-context --current --namespace=argocd'
                 bat ' C:/Users/lokes/argocd.exe login cd.argoproj.io --core'
                // Create or update Argo CD application
                bat 'C:/Users/lokes/argocd.exe app create pipeline --repo https://github.com/lokesh2201013/DevopsProject --path kubeconfig --dest-server https://kubernetes.default.svc --dest-namespace default'
               def podsReady = false
                def maxRetries = 30
                def retryInterval = 60 // seconds

                // Sync Argo CD application
                bat 'C:/Users/lokes/argocd.exe app sync pipeline'
                     sleep 60
                bat 'kubectl apply -f kubeconfig/new.yml -n default'
                bat 'kubectl get pods -n argocd'

                // Check deployments in the default namespace
                bat 'kubectl get deployments -n default'
                bat "echo Current directory: && cd"
            /*  def powershellStatus = powershell script: '''
                    Start-Process -NoNewWindow -FilePath "kubectl" -ArgumentList "port-forward deployment/pipeline 5173:80 -n default" -PassThru
                ''', returnStatus: true
                
                if (powershellStatus == 0) {
                    echo "PowerShell command executed successfully."
                    currentBuild.result = 'SUCCESS'
                    return
                } else {
                    echo "PowerShell command failed."
                    currentBuild.result = 'FAILURE'
                    return
                }*/
                    
                // Send email notification
                mail to: 'lokeshchoraria60369@gmail.com', subject: 'Build Status', body: 'The build has completed.'
                
            }
        }
    }
}
