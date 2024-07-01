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

        stage('Scan Image with Trivy') {
            steps {
                script {
                    bat "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image pipeline:latest --scanners vuln"
                }
            }
        }

        stage('Run Container at port 5173') {
            steps {
                script {
                    bat "docker run -d -p 5173:5173 --name pipeline pipeline:latest"
                }
            }
        }
    }

    post {
        always {
            script {
                bat 'docker stop pipeline'
                bat 'docker rm pipeline'
                 mail to: 'lokeshchoraria60369@gmail.com', subject: 'Build Status', body: 'The build has completed.'
            }
        }
    }
}
