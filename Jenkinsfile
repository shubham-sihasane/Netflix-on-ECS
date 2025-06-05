pipeline {
  agent any

  environment {
    SCANNER_HOME = tool 'sonarscanner'
    APP_NAME = "ElasticApp"
    //DOCKER_REGISTRY = "sihasaneshubham"
    DOCKER_REGISTRY = "680829786414.dkr.ecr.ap-south-1.amazonaws.com"
    DOCKER_REPOSITORY = 'netflix'
    DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
  }

  stages {
    stage('Clone Repository') {
      steps {
        git branch: 'main', url: 'https://github.com/shubham-sihasane/Netflix-on-ECS.git'
      }
    }
    stage('Trivy Filescan') {
      steps {
        sh 'trivy fs --format table -o trivy-fs-report.html .'
      }
    }
    stage('Sonarqube Analysis') {
      steps {
        withSonarQubeEnv('sonarserver') {
          sh '''
            $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=$APP_NAME -Dsonar.projectKey=$APP_NAME -Dsonar.sources=.
            echo $SCANNER_HOME
          '''
        }
      }
    }
    stage("Quality Gate") {
      steps {
        timeout(time: 1, unit: 'HOURS') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Build Image') {
      steps {
        script {
          withDockerRegistry(credentialsId: 'DockerRegistry') {
            sh """
              docker build -t $DOCKER_REGISTRY/$DOCKER_REPOSITORY:$DOCKER_IMAGE_TAG -f Dockerfile .
              docker image tag $DOCKER_REGISTRY/$DOCKER_REPOSITORY:$DOCKER_IMAGE_TAG $DOCKER_REGISTRY/$DOCKER_REPOSITORY:DOCKER_IMAGE_TAG
              docker image tag $DOCKER_REGISTRY/$DOCKER_REPOSITORY:$DOCKER_IMAGE_TAG $DOCKER_REGISTRY/$DOCKER_REPOSITORY:latest
            """
          }
        }
      }
    }
    stage('Image Scan') {
      steps {
        sh 'trivy --severity HIGH,CRITICAL --format table -o trivy-image-report.html image $DOCKER_REGISTRY/$DOCKER_REPOSITORY:$DOCKER_IMAGE_TAG'
      }
    }
    stage('Push Image to ECR') {
      steps {
        sh """
          aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $DOCKER_REGISTRY
          docker image push $DOCKER_REGISTRY/$DOCKER_REPOSITORY:$DOCKER_IMAGE_TAG
          docker image push $DOCKER_REGISTRY/$DOCKER_REPOSITORY:latest
        """
      }
    }
  }

}


        