pipeline {
    agent any

    stages {
        stage('Build') {
          agent {
            docker {
              image 'node:18-alpine'
              reuseNode true
            }
          }
            steps {
               sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm install serve
                npm run build
                ls -la
              '''
            }
        }
         stage('Test') {
          agent {
            docker {
              image 'node:18-alpine'
              reuseNode true
            }
          }
            steps {
             echo 'Test stage'
             sh '''

             if [ -f "build/index.html" ]; then
              echo "File exists."
             else
              echo "File does not exist."
              exit 1 # Fail the pipeline
              fi

             npm test
            '''
            }
        }
          stage('E2E') {
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
              reuseNode true
            }
          }
            steps {
             echo 'E2E stage'
             sh '''
             node_modules/.bin/serve -s build &
             sleep 10
             npx playwright test
            '''
            }
        }
    }
    post {
      always {
        junit 'jest-results/junit.xml'
      }
    }
}
