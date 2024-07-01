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
            environment {
                // Define a variable to store the container ID
                CONTAINER_ID = ''
            }
            steps {
                script {
                    // Run the container and capture the container ID
                    def output = bat(script: 'docker run -d -p 5173:5173 pipeline', returnStdout: true).trim()
                    env.CONTAINER_ID = output
                }
            }
        }
    }

    post {
        always {
            script {
                // Stop and remove the container created by Jenkins
                if (env.CONTAINER_ID) {
                    bat "docker stop ${env.CONTAINER_ID}"
                    bat "docker rm ${env.CONTAINER_ID}"
                }
            }
        }
    }
}
