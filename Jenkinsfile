pipeline {
  agent any
  options { timestamps() }

  environment {
    SERVER_IMAGE = 'amelnechi/mern-server'
    CLIENT_IMAGE = 'amelnechi/mern-client'
    TAG          = "build-${env.BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Docker Build') {
      steps {
        bat 'docker version'
        bat "docker build -t %SERVER_IMAGE%:%TAG% ./Server"
        bat "docker build -t %CLIENT_IMAGE%:%TAG% ./Client"
      }
    }

    stage('Smoke Test') {
      steps {
        bat """
          docker rm -f server_test client_test 2>nul || ver > nul
          docker run -d --name server_test -p 9001:9000 %SERVER_IMAGE%:%TAG%
          docker run -d --name client_test -p 3001:3000 %CLIENT_IMAGE%:%TAG%
          ping -n 8 127.0.0.1 > nul
          curl -s http://localhost:9001/api/health | find "API is running"
          if errorlevel 1 (docker rm -f server_test client_test & exit /b 1)
          curl -I http://localhost:3001 | find "200"
          if errorlevel 1 (docker rm -f server_test client_test & exit /b 1)
          docker rm -f server_test client_test
        """
      }
    }

    stage('Push (Docker Hub)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'USER',
                                          passwordVariable: 'PASS')]) {
          bat """
            echo %PASS%| docker login -u %USER% --password-stdin
            docker tag %SERVER_IMAGE%:%TAG% %SERVER_IMAGE%:latest
            docker tag %CLIENT_IMAGE%:%TAG% %CLIENT_IMAGE%:latest
            docker push %SERVER_IMAGE%:%TAG%
            docker push %SERVER_IMAGE%:latest
            docker push %CLIENT_IMAGE%:%TAG%
            docker push %CLIENT_IMAGE%:latest
          """
        }
      }
    }

    stage('Deploy') {
      steps {
        bat """
          docker compose down
          docker compose up -d --build
          docker ps
        """
      }
    }
  }

  post {
    success { echo 'Build+Test+Push+Deploy OK' }
    failure { echo 'Pipeline KO' }
  }
}