pipeline {
    agent any
    stages {
        stage('Build') {
            when {
                branch 'master'
            }
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker image') {
          when {
            branch 'master'
          }
          steps {
            echo 'Building docker image'
            sh "docker build -tag snirkop/train-schedule ."
          }
        }
        stage('Push Docker image') {
            when {
                branch 'master'
            }
          withCredentials([usernamePassword(credentialsId: 'docker-login', usernameVariable: 'DOCKERUSER', passwordVariable: 'DOCKERPASS')]) {
              steps {
                  sh "docker login -u $DOCKERUSER -p $DOCKERPASS"
                  sh "docker push snirkop/train-schedule:${BUILD_NUMBER}"
              }
          }
        }
        stage ('DeployToProduction') {
          when {
              branch 'master'
          }
          steps {
              input 'Deploy to Production'
              milestone(1)
              withCredentials ([usernamePassword(credentialsId: 'prod-server', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                  script {
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker pull snirkop/train-schedule:${env.BUILD_NUMBER}\""
                      try {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker stop train-schedule\""
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker rm train-schedule\""
                      } catch (err) {
                          echo: 'caught error: $err'
                      }
                      sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker run --restart always --name train-schedule -p 8080:8080 -d <DOCKER_HUB_USERNAME>/train-schedule:${env.BUILD_NUMBER}\""
                  }
              }
          }
        }
    }
}
