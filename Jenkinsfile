pipeline {
  agent any
  
  parameters {
    string name: 'IMAGE_NAME', defaultValue: 'eureka-server'
    string name: 'IMAGE_REGISTRY_ACCOUNT', defaultValue: 'hahajong'
  }

  stages {    
    stage('Test Gradle Project') {
      steps {
          sh './gradlew test --no-daemon'
      }
    }
    stage('Build Gradle Project') {
      steps {
          sh './gradlew build -x test --parallel --no-daemon'
      }
    }
    
    stage('Build Docker Image') {
      steps {
        sh "docker image build -t ${params.IMAGE_NAME} ."
      }
    }

    stage('Remove Docker Image') {
      when {
        not {
            anyOf {
            branch 'main';
            branch 'be-dev'
            }
        }
      }
      steps {
        sh "docker images ${params.IMAGE_NAME} -q | xargs docker rmi -f"
      }
    }


    stage('Tagging Docker Image') {
      when {
        anyOf {
          branch 'main';
          branch 'be-dev'
        }
      }
      steps {
        sh "docker image tag ${params.IMAGE_NAME} ${params.IMAGE_REGISTRY_ACCOUNT}/${params.IMAGE_NAME}:latest"
        sh "docker image tag ${params.IMAGE_NAME} ${params.IMAGE_REGISTRY_ACCOUNT}/${params.IMAGE_NAME}:${BUILD_NUMBER}"
      }
    }


    stage('Publish Docker Image') {
      when {
        anyOf {
          branch 'main';
          branch 'be-dev'
        }
      }
      steps {
        withDockerRegistry(credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/') {
          sh "docker image push --all-tags ${params.IMAGE_REGISTRY_ACCOUNT}/${params.IMAGE_NAME}"
          sh "docker images ${params.IMAGE_NAME} -q | xargs docker rmi -f"
        }
      }
    }

    stage('Update Kubernetes manifests') {
      when {
        anyOf {
          branch 'main';
          branch 'be-dev'
        }
      }
      steps {
            git branch: 'main', credentialsId: 'cicd-sssdev', url: 'https://github.com/sss-develops/application-manifests.git'
            sh "./change-image-tag.sh ${params.IMAGE_REGISTRY_ACCOUNT} ${params.IMAGE_NAME} ${env.BUILD_NUMBER} ${env.WORKSPACE}"
            withCredentials([gitUsernamePassword(credentialsId: 'cicd-sssdev', gitToolName: 'Default')]) {
              sh "git push origin main"
            }
        }
    }
  }
}