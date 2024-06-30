pipeline {
    agent any

    stages {
        stage('Build Image') {
            steps {
                script {
                    bat 'docker build -t pipeline .'
                }
            }
        }
        
        stage('Run Container at port 5173') {
            steps {
                script {
                    bat 'docker run -d -p 5173:5173 pipeline'
                }
            }
        }
    }
    post {
        always {
            // Cleanup: Stop and remove Docker containers after pipeline execution
            script {
                bat 'docker stop (docker ps -aq --filter ancestor=pipeline)'
                bat 'docker rm (docker ps -aq --filter ancestor=pipeline)'
            }
        }
    }
}
