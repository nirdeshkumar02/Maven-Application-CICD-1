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
        stage('Upload Jar File to Nexus'){
            steps{
                script{
                    //  Reading pom.xml file for versioning
                    def readPomVersion = readMavenPom file: 'pom.xml'
                    //  Checking for condition, if artifect is release save it to release repo otherwise put it in snap repo in the nexus
                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "maven-appliction-snapshot" : "maven-appliction-release"
                    //  Upload to nexus
                    nexusArtifactUploader artifacts: 
                    [
                        [
                            artifactId: 'springboot', 
                            classifier: '', 
                            file: 'target/Uber.jar', 
                            type: 'jar'
                        ]
                    ], 
                    credentialsId: 'nexus-auth', 
                    groupId: 'com.example', 
                    nexusUrl: '3.108.194.173:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: "${nexusRepo}",
                    version: "${readPomVersion.version}"
                }
            }
        }
        stage("Docker Build"){
            steps{
                script{
                    sh "docker build -t $JOB_NAME:v1.$BUILD_ID ."
                    sh "docker tag $JOB_NAME:v1.$BUILD_ID nirdeshkumar02/$JOB_NAME:v1.$BUILD_ID"
                    sh "docker tag $JOB_NAME:v1.$BUILD_ID nirdeshkumar02/$JOB_NAME:latest"
                }
            }
        }
        stage("Push Image to DockerHub"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_pass')]) {
                        sh "docker login -u nirdeshkumar02 -p ${docker_pass}"
                        sh "docker push nirdeshkumar02/$JOB_NAME:v1.$BUILD_ID"
                        sh "docker push nirdeshkumar02/$JOB_NAME:latest"
                    }
                }
            }
        }
    }
    post{
        always {
            mail bcc: '', body: "<br>Project: $JOB_NAME <br>Build Number: $BUILD_NUMBER <br> URL: $BUILD_URL", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project Name -> $JOB_NAME", to: "nirdesh.saini@shunyeka.com";
        }
    }
}