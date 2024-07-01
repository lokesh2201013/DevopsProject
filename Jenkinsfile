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
                    bat "docker run -d -p 5173:5173 pipeline"
                }
            }
        }
    }

    post {
        always {
            script {
               def dockerContainers = bat(script: 'docker ps -aq --filter ancestor=pipeline', returnStdout: true).trim()
                    
                    dockerContainers.readLines().each { containerId ->
                        bat "docker stop ${containerId}"
            }
        }
    }
}
