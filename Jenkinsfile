pipeline {
  agent none
  stages {
    stage('worker build') {
      agent {
        docker {
          image 'maven:3.8.4-jdk-8'
          args '-v $HOME/.m2/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Compile worker app'
        dir(path: 'worker') {
          sh 'mvn compile'
        }

      }
    }

    stage('worker test') {
      agent {
        docker {
          image 'maven:3.8.4-jdk-8'
          args '-v $HOME/.m2/root/.m2'
        }

      }
      when {
        changeset '**/worker/**'
      }
      steps {
        echo 'Running Unit Tests on Worker app'
        dir(path: 'worker') {
          sh 'mvn clean test'
        }

      }
    }

    stage('worker package') {
      agent {
        docker {
          image 'maven:3.8.4-jdk-8'
          args '-v $HOME/.m2/root/.m2'
        }

      }
      when {
        branch 'master'
        changeset '**/worker/**'
      }
      steps {
        echo 'Packaging worker app'
        dir(path: 'worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
        }

      }
    }

    stage('worker docker-package') {
      agent any
      when {
        changeset '**/worker/**'
        branch 'master'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker-creds') {
            def workerImage = docker.build("lenodar/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('result build') {
      agent {
        docker {
          image 'node:14.19.0-alpine3.15'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Compile result app'
        dir(path: 'result') {
          sh 'npm install'
        }

      }
    }

    stage('result test') {
      agent {
        docker {
          image 'node:14.19.0-alpine3.15'
        }

      }
      when {
        changeset '**/result/**'
      }
      steps {
        echo 'Running Unit Tests on result app'
        dir(path: 'result') {
          sh 'npm install'
          sh 'npm test'
        }

      }
    }

    stage('result docker-package') {
      agent any
      when {
        changeset '**/result/**'
        branch 'master'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker-creds') {
            def workerImage = docker.build("lenodar/result:v${env.BUILD_ID}", "./result")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('vote build') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Compile vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
        }

      }
    }

    stage('vote test') {
      agent {
        docker {
          image 'python:2.7.16-slim'
          args '--user root'
        }

      }
      when {
        changeset '**/vote/**'
      }
      steps {
        echo 'Running Unit Tests on vote app'
        dir(path: 'vote') {
          sh 'pip install -r requirements.txt'
          sh 'nosetests -v'
        }

      }
    }

    stage('vote docker-package') {
      agent any
      when {
        changeset '**/vote/**'
        branch 'master'
      }
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'docker-creds') {
            def workerImage = docker.build("lenodar/vote:v${env.BUILD_ID}", "./vote")
            workerImage.push()
            workerImage.push("${env.BRANCH_NAME}")
          }
        }

      }
    }

    stage('Sonarqube') {
      agent any
      //when{
      //  branch 'master'
      //}

      environment{
        sonarpath = tool 'SonarScanner'
      }

      steps {
            echo 'Running Sonarqube Analysis..'
            withSonarQubeEnv('sonar-instavote') {
              sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
            }
      }
    }

    stage("Quality Gate") {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
                waitForQualityGate abortPipeline: true
            }
        }
    }
    stage('Deploy to DEV') {
      agent any
      when { 
        branch "master"
      }
      steps {
        sh 'docker-compose up -d'
      }
    }

  }
  post {
    always {
      echo 'Pipline for worker is complete...'
    }

    failure {
      office365ConnectorSend(webhookUrl: 'https://venturecorporate.webhook.office.com/webhookb2/07e4c0f5-3d39-4dce-99a6-a04e14e2de3a@0a34c976-4d3f-4f45-93c4-6f7932cf637d/IncomingWebhook/eaee1e4279dd40aeaca768a4a00bb86f/76a2818f-6409-4b87-b582-8b4a7d0a9826', message: 'The building process failed so hard I cant even tell you right now', status: 'Failed')
    }

    success {
      office365ConnectorSend(webhookUrl: 'https://venturecorporate.webhook.office.com/webhookb2/07e4c0f5-3d39-4dce-99a6-a04e14e2de3a@0a34c976-4d3f-4f45-93c4-6f7932cf637d/IncomingWebhook/eaee1e4279dd40aeaca768a4a00bb86f/76a2818f-6409-4b87-b582-8b4a7d0a9826', message: 'OMG youre such a pro! The build was a complete success', status: 'Success')
    }

  }
}