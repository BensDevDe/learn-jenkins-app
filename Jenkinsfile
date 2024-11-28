pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '640a1281-afe1-4d72-b224-eaeeb6a8f08c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

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
                echo 'Small Change'
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
        //  stage('Test') {
        //   agent {
        //     docker {
        //       image 'node:18-alpine'
        //       reuseNode true
        //     }
        //   }
        //     steps {
        //      echo 'Test stage'
        //      sh '''

        //      if [ -f "build/index.html" ]; then
        //       echo "File exists."
        //      else
        //       echo "File does not exist."
        //       exit 1 # Fail the pipeline
        //       fi

        //      npm test
        //     '''
        //     }
        // }
        //   stage('E2E') {
        //   agent {
        //     docker {
        //       image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //       reuseNode true
        //     }
        //   }
        //     steps {
        //      echo 'E2E stage'
        //      sh '''
        //      node_modules/.bin/serve -s build &
        //      sleep 10
        //      npx playwright test --reporter=html
        //     '''
        //     }
        // }
        stage('Run Tests') {
            parallel {
                stage('Unit-Test') {
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
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
                  npx playwright test --reporter=html
                  '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
               npm install netlify-cli
               node_modules/.bin/netlify netlify --version
               echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
               node_modules/.bin/netlify status
               node_modules/.bin/netlify deploy --dir=build
              '''
            }
        }
        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
               npm install netlify-cli
               node_modules/.bin/netlify netlify --version
               echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
               node_modules/.bin/netlify status
               node_modules/.bin/netlify deploy --dir=build --prod
              '''
            }
        }
        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://stellular-croissant-1af89e.netlify.app/'
            }
            steps {
                echo 'E2E stage'
                sh '''
              npx playwright test --reporter=html
            '''
            }
            post {
                    always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
            }
        }
    }
}
