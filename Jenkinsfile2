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
                    for( int i=0 ; i<maxtries ;i++)
                    {
                        def podStatus = bat(script:'kubectl get pods -n argocd --field-selector=status.phase=Running', returnStatus: true)
                        if(podStatus==0)
                        {
                            echo "Argocd ready"
                            podsready=true
                            break
                        }
                        else{
                              echo "Waiting for pods to be ready in Argo CD namespace..."
                        sleep retry
                        }

                    }
                      if (!podsready) {
                    error "Timed out waiting for pods to be ready in Argo CD namespace."
                }
                 
                // Set current context to the Argo CD namespace (if needed)
                bat 'kubectl config set-context --current --namespace=argocd'
                 bat ' C:/Users/lokes/argocd.exe login cd.argoproj.io --core'
                // Create or update Argo CD application
                bat 'C:/Users/lokes/argocd.exe app create pipeline --repo https://github.com/lokesh2201013/DevopsProject --path kubeconfig --dest-server https://kubernetes.default.svc --dest-namespace default'
               def podsReady = false
                def maxRetries = 30
                def retryInterval = 60 // seconds

                for (int i = 0; i < maxRetries; i++) {
                    def podStatus = sh(script: 'kubectl get pods -n default --field-selector=status.phase=Running | findstr "pipeline"', returnStatus: true)
                    if (podStatus == 0) {
                        echo "Pipeline pod in default namespace is ready."
                        podsReady = true
                        break
                    } else {
                        echo "Waiting for pipeline pod to be ready in default namespace..."
                        sleep retryInterval
                    }
                }

                if (!podsReady) {
                    error "Timed out waiting for pipeline pod to be ready in default namespace."
                }

                // Sync Argo CD application
                bat 'C:/Users/lokes/argocd.exe app sync pipeline'

                // Check pods in the Argo CD namespace
                bat 'kubectl get pods -n argocd'

                // Check deployments in the default namespace
                bat 'kubectl get deployments -n default'

                // Send email notification
                mail to: 'lokeshchoraria60369@gmail.com', subject: 'Build Status', body: 'The build has completed.'
            }
        }
    }
}