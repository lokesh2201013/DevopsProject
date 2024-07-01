pipeline {
    agent any

    stages {
        stage('Build Image') {
            steps {
                script {
                    bat "docker build -t pipeline ."
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    bat "kubectl apply -f deployment.yaml"
                }
            }
        }

        stage('Run Container at port 5173') {
            steps {
                script {
                    // Run Docker container
                    bat "docker run -d -p 5173:5173 --name pipeline lokesh220/pipeline:latest"
                }
            }
        }
    }

    post {
        always {
            script {
                // Stop and remove Docker container
                bat 'docker stop pipeline'
                bat 'docker rm pipeline'
                
                // Send email notification
                mail to: 'lokeshchoraria60369@gmail.com', subject: 'Build Status', body: 'The build has completed.'
            }
        }
    }
}
