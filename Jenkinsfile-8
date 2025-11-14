pipeline {
  agent any

  parameters {
    string(name: "App_Version", description: "provide application version")
  }

  environment {
    DOCKERHUB_CREDENTIALS=credentials("dockerhub")
  }

  stages {
    stage("Repo Clone") {
      steps {
        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/shivagande26/DataStore.git']])
      }
    }
    stage("Maven Build") {
      steps {
        sh """
            echo "-------- Building Application --------"
            mvn clean package
            echo "------- Application Built Successfully --------"
        """
      }
    }
    stage("Maven Test") {
      steps {
        sh """
          echo "-------- Executing Testcases --------"
          mvn test
          echo "-------- Testcases Execution Complete --------"
        """
      }
    }
    stage("Artifact Store") {
      steps {
        sh """
          echo "-------- Pushing Artifacts To S3 --------"
          aws s3 cp ./target/*.jar s3://datastore-artefact-store-apps/
          echo "-------- Pushing Artifacts To S3 Completed --------"
        """
      }
    }
    stage("Docker Image Build") {
      steps {
        sh """
          echo "-------- Building Docker Image --------"
          docker build -t datastore:"${App_Version}" .
          echo "-------- Image Successfully Built --------"
        """
      }
    }
    stage("Docker Image Scan") {
      steps {
        sh """
          echo "-------- Scanning Docker Image --------"
          trivy image datastore:"${App_Version}"
          echo "-------- Scanning Docker Image Complete --------"
        """
      }
    }
    stage("Docker Image Tag") {
      steps{
        sh """
          echo "-------- Tagging Docker Image --------"
          docker tag datastore:"${App_Version}" 8072388539/datastore:"${App_Version}"
          echo "-------- Tagging Docker Image Completed."
        """
      }
    }
    stage("Loggingin & Pushing Docker image") {
      steps {
        sh """
          echo "-------- Logging To DockerHub --------"
          docker login -u $DOCKERHUB_CREDENTIALS_USR --password $DOCKERHUB_CREDENTIALS_PSW
          echo "-------- DockerHub Login Successful --------"

          echo "-------- Pushing Docker Image To DockerHub --------"
          docker push 8072388539/datastore:"${App_Version}"
          echo "-------- Docker Image Pushed Successfully --------"
        """
      }
    }
    stage("cleanup") {
      steps {
        sh """
           echo "-------- Cleaning Up Jenkins Machine --------"
           docker image prune -a -f
           echo "-------- Clean Up Successful --------"
        """
      }
    }
    stage("Deployment Acceptance") {
      steps {
        input 'Trigger Down Stream Job'
      }
    }
  }
}
