pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('BuildDocker') {
            when {
                branch 'master'
            }
            steps {
                echo 'Running build Docker'
                script {
                    def customImage = docker.build("aiep/train-schedule:${env.BUILD_ID}")
                }
            }
        }
        stage('PushDocker') {
            when {
                branch 'master'
            }
            steps {
                echo 'Pushing Docker Image to Registry'
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        customImage.push("${env.BUILD_NUMBER}")
                        customImage.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull aiep/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d aiep/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
