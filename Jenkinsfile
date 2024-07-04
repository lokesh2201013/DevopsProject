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
                    bat 'docker stop pipeline'
                    bat 'docker rm pipeline'
                }
            }
        }
    }
    stage('Argo cluster configrations')
    {
        steps
        script{
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
                    sleep 120
                      bat 'kubectl get pods -n argocd'

                } else {
                    echo "Argo CD namespace exists."
                } 
             bat 'kubectl config set-context --current --namespace=argocd'
            
               //login in argo cd
                bat ' C:/Users/lokes/argocd.exe login cd.argoproj.io --core'
            
                // Create or update Argo CD application
                bat 'C:/Users/lokes/argocd.exe app create pipeline --repo https://github.com/lokesh2201013/DevopsProject --path kubeconfig --dest-server https://kubernetes.default.svc --dest-namespace default'
            // Sync Argo CD application
                bat 'C:/Users/lokes/argocd.exe app sync pipeline'
                 sleep 30
             // Check deployments in the default namespace
                bat 'kubectl get deployments -n default'
        }
    }
    stage('parellel execution')
    {
        parellel
        {
            stage('Deploying and port forwarding the app')
            {
                 def powershellStatus = powershell script: '''
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
                }
            }
            stage('killing the task')
            { 
                    // sleep for 30s to see the output 
                    sleep 30
                    def taskid = bat(script: 'netstat -ano | findstr "5173"', returnStdout: true).trim()
                     
                    // Extract PID from netstat output
                    def pid = taskid.tokenize()[4]

                    // Kill the process with the extracted PID using taskkill
                    bat "taskkill /F /PID ${pid}"
            }
        }
    }
    post {
        always {
            script {
                mail to: 'lokeshchoraria60369@gmail.com', subject: 'Build Status', body: 'The build has completed.'
                
            }
        }
    }
}
