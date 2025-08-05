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
metadata:
  name: example-pod
spec:
  serviceAccountName: jenkins 
  securityContext:                # Pod级别securityContext
    fsGroup: 1000                # 正确放置fsGroup
    runAsUser: 1000
    runAsGroup: 1000
  containers:
  - name: kubectl
    image: bitnami/kubectl:1.27.4
    command:
    - sleep
    - "3600"
    tty: true
    securityContext:             # 容器级别securityContext
      runAsUser: 1000
      runAsGroup: 1000
"""
    }
  }

  parameters {
    choice(
      name: 'choices', 
      choices: ['deploy blue', 'deploy green', 'switch traffic', 'rollout blue','delete all'], 
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
              sh """
                cat svc-prod.yaml
                sed -i 's/app: demo-blue/app: demo-green/' svc-prod.yaml
                cat svc-prod.yaml
                
                git config user.email "jenkins@example.com"
                git config user.name "Jenkins CI"
                
                git add svc-prod.yaml
                
                git commit -m "Switch traffic: update svc-prod.yaml selector from demo-blue to demo-green" || echo "No changes to commit"
                
                git push origin HEAD:main
              """
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
              // sh "kubectl -n blue-green patch svc demo-blue-svc -p '{\"spec\":{\"selector\":{\"app\":\"demo-blue\"}}}'"
              // 用 sed 替换 selector 中的 app 名
              sh """
                cat svc-prod.yaml
                sed -i 's/app: demo-green/app: demo-blue/' svc-prod.yaml
                cat svc-prod.yaml
              """
            } else {
              echo "Skipping rollout blue"
            }
          }
        }
      }
    }

      stage('delete all') {
      steps {
        container('kubectl') {
          script {
            if (params.choices == 'delete all') {
              sh "kubectl -n blue-green delete -f ./"
            } else {
              echo "Skipping rollout blue"
            }
          }
        }
      }
    }
  }
}
