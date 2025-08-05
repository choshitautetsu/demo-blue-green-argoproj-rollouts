pipeline {
  options {
    skipDefaultCheckout()
  }
  agent {
    kubernetes {
      cloud 'gke'
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins 
  containers:
  - name: kubectl
    image: bitnami/kubectl:1.27.4
    command:
    - sleep
    - "3600"
    tty: true
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
      fsGroup: 1000
"""
    }
  }

  parameters {
    choice(
      name: 'choices', 
      choices: ['deploy blue', 'deploy green', 'switch traffic', 'rollout blue'], 
      description: 'pick one'
    )
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Deploy Blue') {
      steps {
        container('kubectl') {
          script {
            if (params.choices == 'deploy blue') {
              sh 'kubectl apply -f deploy-blue.yaml'
              sh 'kubectl apply -f svc-prod.yaml'
              sh 'kubectl rollout status deployment/demo-blue -n blue-green --timeout=120s'
            } else {
              echo "Skipping deploy blue"
            }
          }
        }
      }
    }

    stage('Deploy Green') {
      steps {
        container('kubectl') {
          script {
            if (params.choices == 'deploy green') {
              sh 'kubectl apply -f deploy-green.yaml'
              sh 'kubectl apply -f svc-dev.yaml'
              sh 'kubectl rollout status deployment/demo-green -n blue-green --timeout=120s'
            } else {
              echo "Skipping deploy green"
            }
          }
        }
      }
    }

    stage('Switch Traffic') {
      steps {
        container('kubectl') {
          script {
            if (params.choices == 'switch traffic') {
              sh "kubectl -n blue-green patch svc demo-blue-svc -p '{\"spec\":{\"selector\":{\"app\":\"demo-green\"}}}'"
            } else {
              echo "Skipping switch traffic"
            }
          }
        }
      }
    }

    stage('Rollout Blue') {
      steps {
        container('kubectl') {
          script {
            if (params.choices == 'rollout blue') {
              sh "kubectl -n blue-green patch svc demo-blue-svc -p '{\"spec\":{\"selector\":{\"app\":\"demo-blue\"}}}'"
            } else {
              echo "Skipping rollout blue"
            }
          }
        }
      }
    }
  }
// }
