pipeline {
    agent any
    environment {
        PATH = "/opt/apachemaven/bin:$PATH"
    }
    stages {
        stage('SCM Checkout') {
            steps{
            git branch: 'main', credentialsId: 'GITHUB', url: 'git@github.com:jaggugithub/JavaWebApp.git'
            }
        }
        stage('Build Code') {
            steps {  
                sh "mvn clean install"
            }
        }
        stage('Deploy Code') {
            steps {
                sshagent(['DockerServer']) {
                    // This is to Copy a file From Jenkins Server to Docker Server
                    sh "scp -o StrictHostKeyChecking=no target/JWA.war ubuntu@18.132.209.49:/home/ubuntu/docker"
                }
                
            }
        }
        stage('Build Image') {
            steps {
                sshagent(['DockerServer']) {
                    // This is to build a docker image on ubuntu machine(AWS)(Name:Docker Server) from jenkins pipeline
                    sh """ssh -tt -o StrictHostKeyChecking=no ubuntu@18.132.209.49 << EOF
                        cd /home/ubuntu/docker
                        docker build -t jaggu199/webapp:$BUILD_NUMBER . 
                        exit
                        EOF"""
                }
                
            }
        }
        stage('Run Container') {
            steps {
                sshagent(['DockerServer']) {
                    // This is to run a docker container which is on ubuntu machine(AWS)(Name:Docker Server) from jenkins pipeline
                    sh """ssh -tt -o StrictHostKeyChecking=no ubuntu@18.132.209.49 << EOF
                        cd /home/ubuntu/docker
                        docker stop webappcontainer
                        docker rm webappcontainer
                        docker run -d -p 8090:8080 --name webappcontainer jaggu199/webapp:$BUILD_NUMBER
                        exit
                        EOF"""
                }
                
            }
        }
    }
    // post {
    //     success {
    //         mail to: 'team@example.com',
    //         subject: "Success Pipeline: ${currentBuild.fullDisplayName}",
    //         body: "Your Build Is Successfull ${env.BUILD_URL}"
    //     }
    // }
}