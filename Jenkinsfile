pipeline {
    agent any

    stages {
        // Your stages here
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

    options {
        buildDiscarder(logRotator(numToKeepStr: '30'))
    }
}
