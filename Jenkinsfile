#!/usr/bin/env groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'rest-demo'
        APP_VERSION = '1.0.0' // should be changed to the parameter value
        APP_RUNTIME = 'develop'

        // aws cli credentials
        ACCESS_KEY = credentials('aws-dvo-access-key')
        SECRET_KEY = credentials('aws-dvo-secret-key')
        REGION = 'ap-northeast-1'
    }

    stages {
        /*
         * Build Module (war file)
         */
        stage('Build Module') {
            steps {
                sh './sample/template/jenkins/run_maven_build.sh'
            }
        }

        /*
         * Build Container Image
         */
        stage('Build Image') {
            environment {
                BUILD_ARTIFACT = 'samples/template/target/rest-demo.war'
            }
            steps {
                sh "cp ${BUILD_ARTIFACT} samples/template/container/src"
                dir('samples/template/container/src') {
                    sh "docker build -t ${APP_NAME}:${APP_VERSION} ."
                }
            }
        }

        /*
         * Register Container Image to ECR
         */ 
        stage('Register Image') {
            environment {
                ECR_ENDPT = '572065744477.dkr.ecr.ap-northeast-1.amazonaws.com'
                DOCKER_LOGIN_PWD = sh(script: '''
                    docker run --rm \
                        -e AWS_ACCESS_KEY_ID=$ACCESS_KEY \
                        -e AWS_SECRET_ACCESS_KEY=$SECRET_KEY \
                        amazon/aws-cli:latest ecr get-login-password --region $REGION
                ''', returnStdout: true).trim()
            }
            steps {
                sh 'docker login -u AWS -p $DOCKER_LOGIN_PWD $ECR_ENDPT'

                // version tag
                sh 'docker tag $APP_NAME:$APP_VERSION $ECR_ENDPT/$APP_NAME:$APP_VERSION'
                sh 'docker push $ECR_ENDPT/$APP_NAME:$APP_VERSION'

                // runtime tag
                sh 'docker tag $APP_NAME:$APP_VERSION $ECR_ENDPT/$APP_NAME:$APP_RUNTIME'
                sh 'docker push $ECR_ENDPT/$APP_NAME:$APP_RUNTIME'

                sh 'docker logout'
            }
        }

        /*
         * Rollout (update service)
         */ 
        stage('Rollout') {
            environment {
                CLUSTER_NAME = 'DEV_CLUSTER'
                SERVICE_NAME = 'DEV_SERVICE'
            }
            steps {
                // update ecs service with force new deployment option
                sh '''
                    docker run --rm \
                        -e AWS_ACCESS_KEY_ID=$ACCESS_KEY \
                        -e AWS_SECRET_ACCESS_KEY=$SECRET_KEY \
                        amazon/aws-cli:latest ecs update-service --cluster $CLUSTER_NAME --service $SERVICE_NAME --force-new-deployment
                '''
            }
        }
    }
}