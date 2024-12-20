/* groovylint-disable LineLength */

pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '640a1281-afe1-4d72-b224-eaeeb6a8f08c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        // stage('Docker') {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }
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
                npm run build
                ls -la
              '''
            }
        }
                stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            environment {
                AWS_S3_BUCKET = 'learn-jenkins-202412020917'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    aws s3 sync build s3://$AWS_S3_BUCKET
                '''
                }
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
        stage('Tests') {
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
                            // image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        echo 'E2E stage'
                        sh '''
                            serve -s build &
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
                    // image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                echo 'Deploy Staging'
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                    always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Stage E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
            }
        }
         //stage('Approval') {
           // steps {
               // input message: 'Ready to deploy?', ok: 'Yes, i am sure I want to deploy!'
            //}
        //}
        // stage('Deploy prod') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //        npm install netlify-cli
        //        node_modules/.bin/netlify netlify --version
        //        echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //        node_modules/.bin/netlify status
        //        node_modules/.bin/netlify deploy --dir=build --prod
        //       '''
        //     }
        // }
        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://stellular-croissant-1af89e.netlify.app/'
            }
            steps {
                echo 'Deploy Prod'
                sh '''
                node --version
                netlify netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --prod
                npx playwright test --reporter=html
            '''
            }
            post {
                    always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
            }
        }
    }
}
