pipeline {
    agent any
    
    tools {
        maven 'M3'
    }
    
    stages {
        stage('Cleaning workspace'){
            steps {
                sh 'echo "---=--- Cleaning Stage ---=---"'
                sh 'mvn clean'
                script {
                    try {
                        sh 'docker stop myboot && docker rm myboot'
                    } catch (Exception e) {
                        sh 'echo "---=--- No container to remove ---=---"'
                    }                    
                }
            }
        }
        stage('Checkout'){
            steps {
                sh 'echo "---=--- Checkout ---=---"'
                git branch: 'main', url: 'https://github.com/Formation67/eql-boot.git'
            }
        }
        stage('Compile'){
            steps {
                sh 'echo "---=--- Checkout ---=---"'
                sh 'mvn compile'
            }
        }
        stage('Test'){
            steps {
                sh 'echo "---=--- Test ---=---"'
                sh 'mvn test'
            }
        }
        stage('Package'){
            steps {
                sh 'echo "---=--- Package ---=---"'
                sh 'mvn package -DskipTests'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            } 
        }
    stage('SSH transfert') {
        steps {
            script {
                sshPublisher(publishers: [
                    sshPublisherDesc(configName: 'ec2-host', transfers:[
                        sshTransfer(
                          execCommand: '''
                                echo "-=- Cleaning project -=-"
                                sudo docker stop myboot  || true
                                sudo docker rm myboot || true
                                sudo docker rmi mm167/myboot:1.0 || true
                            '''
                        ),
                        sshTransfer(
                            sourceFiles:"target/*.jar",
                            removePrefix: "target",
                            remoteDirectory: "//home//ec2-user",
                            execCommand: "ls /home/ec2-user"
                        ),
                        sshTransfer(
                            sourceFiles:"Dockerfile",
                            removePrefix: "",
                            remoteDirectory: "//home//ec2-user",
                            execCommand: '''
                                cd //home//ec2-user;
                                sudo docker build -t mm167/myboot:1.0 .; 
                                sudo docker run -d --name myboot -p 8180:8180 mm167/myboot:1.0;
                            '''
                        )
                    ])
                ])                
            }
        }
    }
    }
}
