pipeline {
  agent {
    kubernetes {
        yamlFile "./k8s/build.yaml"
    }
  }

  environment {
      SERVICE = "casdoor"
      PROJECT = "halocom-io/${SERVICE}"
      GITHUB_REPO = "https://github.com/${PROJECT}"
      REGISTRY = "registry.digitalocean.com/halocom"
  }

  stages {

    stage('Checkout Source') {
      when {
          branch 'release/*'
      }

      steps {
          checkout scm
          script {
              env.VERSION = env.BRANCH_NAME.split("/")[1]
          }
      }
    }
    
    stage("Build image") {
          environment {
              IMAGE_TAG = '${REGISTRY}/${PROJECT}:${VERSION}'
          }
          steps {
              sh 'echo "${BRANCH_NAME}"'
              container('docker') {
                  script {
                      myapp = docker.build("${IMAGE_TAG}")
                  }
              }
          }
      }
    
    stage("Push image") {
          steps {
              container('docker') {
                  script {
                      docker.withRegistry('https://${REGISTRY}', 'do-registry-id') {
                              myapp.push()
                      }
                  }
              }
          }
     }

    stage('Deploy') {
       environment {
           NAMESPACE = "apisix"
           IMAGE_TAG = "${REGISTRY}/${PROJECT}:${VERSION}"
       }
       steps {
           timeout(10) {
             withKubeConfig([credentialsId: 'k8s-config-file']) {
               sh """
                   sed -i -e 's|{{SERVICE}}|${SERVICE}|g; s|{{IMAGE_TAG}}|${IMAGE_TAG}|g' ./k8s/deployment.base.yaml
                   kubectl apply -f ./k8s/deployment.base.yaml -n ${NAMESPACE} 
                   kubectl rollout status deployment/${SERVICE} -n ${NAMESPACE}
               """
             }
           }
       }
    }
  }
}
