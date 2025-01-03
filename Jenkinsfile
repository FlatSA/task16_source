pipeline {
  agent any

  environment {
    LAST_COMMIT_HASH=''
  }

  stages {

    stage('Build Docker Image') {
      steps {
        sh """
          LAST_COMMIT_HASH=$(git rev-parse HEAD)
          docker login -u $DOCKER_USER -p $DOCKER_PASS
          docker build --no-cache -t flat1337/apache-back apache-server
        """ 
      }
    }

    stage('Push Docker Image') {
      steps {
        sh """
          docker login -u $DOCKER_USER -p $DOCKER_PASS
          docker push flat1337/apache-back:$LAST_COMMIT_HASH
        """
      }
    }

    stage('Deploy to Remote Instance') {
      steps {
        script {
          def command = """
            docker login -u $DOCKER_USER -p $DOCKER_PASS
            if [[ "\$(docker ps -aq -f name=apache-server)" ]]; then
              docker stop apache-server
              docker rm apache-server
            fi

            docker pull flat1337/apache-back:$LAST_COMMIT_HASH

            docker run -itd --name apache-server -p 80:80 flat1337/apache-back:$LAST_COMMIT_HASH

            docker image prune -af
          """
          sshPublisher(publishers: [
            sshPublisherDesc(
              configName: "apache",
              transfers: [
                sshTransfer(
                  sourceFiles: '',
                  execCommand: command,
                  removePrefix: '',
                  remoteDirectory: ''
                )
              ],
              usePromotionTimestamp: false,
              verbose: true
            )
          ])
        }
      }
    }
  }

  post {
    success {
      echo "pipeline succeeded"
    }
    failure {
      echo "pipeline failed"
    }
  }
}
