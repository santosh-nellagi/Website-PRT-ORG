pipeline {
  agent {
    docker {
      image 'santosh-nellagi/demo:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/santosh-nellagi/Website-PRT-OR'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'cd mvn clean package'
      }
    }
    
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "santosh/ultimate-cicd:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "https://github.com/santosh-nellagi/Website-PRT-OR/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'cd Website && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "project"
            GIT_USER_NAME = "santosh-nellagi"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "santoshnellagi740@gmail.com"
                    git config user.name "Santosh Nellagi"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" https://github.com/santosh-nellagi/Website-PRT-OR/deployment.yml
                    git add https://github.com/santosh-nellagi/Website-PRT-OR/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
