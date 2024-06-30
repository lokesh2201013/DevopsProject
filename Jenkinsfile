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
}
