pipeline{
    agent any
    stages{
        stage('Git Checkout'){
            steps{
                script{
                    git branch: 'master', url: 'https://github.com/nirdeshkumar02/Maven-Application-CICD-1.git'
                }
            }
        }
        stage('Unit Testing'){
            steps{
                script{
                    sh 'mvn test'
                }
            }
        }
    }
}