pipeline {

  agent none

  stages{
      stage("build-worker"){
        when{
            changeset "**/worker/**"
          }

        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }

        steps{
          echo 'Compiling worker app..'
          dir('worker'){
            sh 'mvn compile'
          }
        }
      }
      stage("test-worker"){
        when{
          changeset "**/worker/**"
        }
        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }
        steps{
          echo 'Running Unit Tets on worker app..'
          dir('worker'){
            sh 'mvn clean test'
           }

          }
      }
      stage("package-worker"){
        when{
          branch 'master'
          changeset "**/worker/**"
        }
        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
          }
        }
        steps{
          echo 'Packaging worker app'
          dir('worker'){
            sh 'mvn package -DskipTests'
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
          }

        }
      }
      stage('docker-package-worker'){
          agent any
          when{
            changeset "**/worker/**"
            branch 'master'
          }
          steps{
            echo 'Packaging worker app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                  def workerImage = docker.build("zavrelmi/worker:v${env.BUILD_ID}", "./worker")
                  workerImage.push()
                  workerImage.push("${env.BRANCH_NAME}")
                  workerImage.push("latest")
              }
            }
          }
      }
      stage("build-vote"){
              agent{
                      docker{
                              image 'python:2.7-alpine'
                              args '--user root'
                      }
              }

              when{ changeset "**/vote/**"
              }
              steps{
                      echo 'Compiling vote app..'
                      dir('vote'){
                      sh 'pip install -r requirements.txt'
                      }
              }
      }
      stage("test-vote"){
              agent{
                      docker{
                              image 'python:2.7-alpine'
                              args '--user root'
                      }
              }

              when{ changeset "**/vote/**"
              }
              steps{
                      echo 'Running Unit Tets on vote app'
                      dir('vote'){
                              sh 'pip install -r requirements.txt'
                              sh 'nosetests -v'
                      }
              }
      }
      stage('docker-package-vote'){
          agent any
          when{
            changeset "**/vote/**"
            branch 'master'
          }
          steps{
            echo 'Packaging vote app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                  def workerImage = docker.build("zavrelmi/vote:v${env.BUILD_ID}", "./vote")
                  workerImage.push()
                  workerImage.push("${env.BRANCH_NAME}")
                  workerImage.push("latest")
              }
            }
          }
      }
      stage("build-result"){
              agent{
                docker {
                      image 'node:8.16.0-alpine'
                      }
              }

              when{ changeset "**/result/**"
              }
              steps{
                      echo 'Compiling result app..'
                      dir('result'){
                      sh 'npm install'
                      }
              }
      }
      stage("test-result"){
              agent{
                      docker {
                              image 'node:8.16.0-alpine'
                              }
                      }
              when{ changeset "**/result/**"
              }
              steps{
                      echo 'Running Unit Tets on result app'
                      dir('result'){
                              sh 'npm install'
                              sh 'npm test'
                      }
              }
      }
      stage('docker-package-result'){
          agent any
          when{
            changeset "**/result/**"
            branch 'master'
          }
          steps{
            echo 'Packaging result app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                  def workerImage = docker.build("zavrelmi/result:v${env.BUILD_ID}", "./result")
                  workerImage.push()
                  workerImage.push("${env.BRANCH_NAME}")
                  workerImage.push("latest")
              }
            }
          }
      }
      stage('deploy to dev'){
              agent any
              when{
                branch 'master'
              }
              steps{
                echo 'Deploy instavote app with docker compose'
                sh 'docker-compose up -d'
              }
          }
}
        post{
                always{
                        echo 'Building multibranch pipeline for worker is completed..'
                }
        failure{
        slackSend (channel: "jenkins-testing", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)") }
        success{
        slackSend (channel: "jenkins-testing", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)") }

        }
}

