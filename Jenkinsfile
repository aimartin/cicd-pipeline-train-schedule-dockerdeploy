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
            steps {
                echo 'Running build Docker'
                script {
                    def customImage = docker.build("aiep/train-schedule:${env.BUILD_ID}")
                }
            }
        }
        stage('PushDocker') {
            steps {
                echo 'Pushing Docker Image to Registry'
                withDockerRegistry([ credentialsId: "docker_hub_login", url: "" ]) {
                  // following commands will be executed within logged docker registry
                  sh "docker push aiep/train-schedule:${env.BUILD_ID}"
                  //customImage.push()
                }
            }
        }
    }
}
