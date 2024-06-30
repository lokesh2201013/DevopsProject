pipeline
{
    agent any

    stages
    {
        stage('Build Image')
        {
            steps
            {
                script
                {
                    sh "docker build -t pipeline ."
                }
            }
        }
        
        stage('Run Container at port 5173')
        {
            steps{
                script{
                    sh "docker run -d -p 5173:5173 pipeline"
                }
            }
        }
    }
    post {
        always {
            // Cleanup: Stop and remove Docker containers after pipeline execution
            script {
                sh "docker stop \$(docker ps -aq --filter ancestor=pipeline)"
                sh "docker rm \$(docker ps -aq --filter ancestor=pipeline)"
            }
        }
    }
}