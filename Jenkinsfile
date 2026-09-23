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
      stage('Debug') {
         steps {
            sh '''
                  pwd
                  ls -la
                  find . -name Dockerfile -type f
            '''
         }
      }
      stage('Docker Push') {
         steps {
            echo "Running in $WORKSPACE"
            dir("$WORKSPACE/azure-vote") {
               script {
                  docker.withRegistry('', 'dockerhub-cred') {
                     def image = docker.build('adamko034/jenkins-course:2026_3')
                     image.push()
                  }
               }
            }
         }
      }
      stage('Docker Scan with Grype') {
         steps {
            grypeScan autoInstall: true, repName: 'grypeReport_${JOB_NAME}_${BUILD_NUMBER}.txt', scanDest: 'registry:adamko034/jenkins-course:2026_3'
         }
      }
   }
   post {
      always {
         sh(script: 'docker compose down')
         recordIssues(
            tools: [grype()],
            aggregatingResults: true,
         )
      }
   }
}
