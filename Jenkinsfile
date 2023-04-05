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
        stage('Integration Testing'){
            steps{
                script{
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }
        stage('Build Application'){
            steps{
                script{
                    sh 'mvn clean install'
                }
            }
        }
        stage('SAST Analysis with SonarQube'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-creds') {
                        sh 'mvn clean package sonar:sonar'
                    }
                }
            }
        }
        stage('SonarQube Quality Gate Status'){
            steps{
                script{
                    // If Quality Gate Status is failed then pipeline should abort
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar-creds'
                }
            }
        }
    }
}