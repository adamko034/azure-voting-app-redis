pipeline {
   agent any

   stages {
      stage('Verify Branch') {
         steps {
            echo "$GIT_BRANCH"
         }
      }
      stage('Docker Build') {
         steps {
            sh(script: 'docker compose build')
         }
      }
      stage('Start App') {
         steps {
            sh(script: 'docker compose up -d')
         }
      }
      stage('Run Tests') {
         steps {
            sh(script: 'pytest ./tests/test_sample.py')
         }
         post {
            success {
               echo "Tests passed! :)"
            }
            failure {
               echo "Tests failed :("
            }
         }
      }
      stage('Docker Push') {
         steps {
            echo "Runnning in $WORKSPACE"
            dir("$WORKSPACE/azure-vote") {
               script {
                  docker.withRegistry('', 'dockerhub-cred') {
                     def image = docker.build('adamko034/jenkins-course:0.0.1')
                     image.push()
                  }
               }
            }
         }
      }
      stage('QA Deploy') {
         environment {
            KUBECONFIG = credentials('minikube-kubeconfig')
         }
         when {
            branch 'feature/k8s-deploy'
         }
         steps {
            sh '''
               helm upgrade --install azure-vote ./helm/azure-vote \
                 -n azure-vote-qa \
                 -f ./helm/azure-vote/values-qa.yaml
            '''
         }
      }
      stage('Approve Deploy to PROD') {
         when {
            branch 'feature/k8s-deploy'
         }
         options {
            timeout(time: 1, unit: 'HOURS')
         }
         steps {
            input message: "Deploy to PROD?"
         }
      }
      stage('PROD Deploy') {
         environment {
            KUBECONFIG = credentials('minikube-kubeconfig')
         }
         when {
            branch 'feature/k8s-deploy'
         }
         steps {
            sh '''
               helm upgrade --install azure-vote ./helm/azure-vote \
                 -n azure-vote-prod \
                 -f ./helm/azure-vote/values-prod.yaml
            '''
         }
      }
   }
   post {
      always {
         sh(script: 'docker compose down')
      }
   }
}
