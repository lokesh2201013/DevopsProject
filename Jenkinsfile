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

        stage('Run Container at port 5173') {
            steps {
                script {
                    bat "docker run -d -p 5173:5173 --name pipeline pipeline"
                }
            }
        }
    }

    post {
        always {
            script {
                bat 'docker stop pipeline'
                bat 'docker rm pipeline'
            }
        }
    }
}
