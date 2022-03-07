pipeline {
  agent any

  stages {
    stage('Checkout Source') {
      steps {
        git url:'https://github.com/rpfreitas111/k8s-pipeline-jenkins.git', branch:'main'
      }
    }

    stage('Build Image') {
      steps {
        script {
            dockerapp = docker.build("rpfreitas111/k8s-pipeline-jenkins:${env.BUILD_ID}", '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
        }
      }
    }

    stage('Push Image') {
      steps {
        script{
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            dockerapp.push('latest')
            dockerapp.push("${env.BUILD_ID}")
          }
        }
      }
    }

    stage('Deploy kubernetes') {
      agent {
        kubernetes {
          cloud 'kubernetes-microk8s'
        }
      }
      environment {
        tag_version = "${env.BUILD_ID}"
      }
      steps {
        script {
          sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api/api-deployment.yaml'
          sh 'cat ./k8s/api/api-deployment.yaml'
          kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
        }
      }
    }

  }
}
