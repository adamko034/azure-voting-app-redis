pipeline {
   agent any

   environment { 
      DOCKER_IMAGE = 'adamko034/jenkins-course:2026_2' 
      CLAIR_URL = 'http://localhost:6060' 
   }

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
                     def image = docker.build("${DOCKER_IMAGE}")
                     image.push()
                  }
               }
            }
         }
      }
      stage('Wait for Clair') { 
         steps { 
            sh ''' 
               echo "Waiting for Clair..." 
               for i in $(seq 1 30); do 
                  if docker exec clair curl -fs http://localhost:6060/health > /dev/null 2>&1; then 
                     echo "Clair is ready" exit 0 
                  fi 
                     echo "Clair not ready yet..." 
                     sleep 5 
                  done 
                  
                  echo "Clair did not become ready" 
                  docker logs clair --tail 100 
                  exit 1 
               ''' 
         } 
      } 
      stage('Clair Scan') { 
         steps { 
            sh ''' 
               echo "Scanning ${DOCKER_IMAGE} with Clair..." 
               docker exec clair clairctl report \ 
                  --host ${CLAIR_URL} \ 
                  ${DOCKER_IMAGE} 
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
