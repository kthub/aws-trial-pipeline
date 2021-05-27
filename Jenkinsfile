#!/usr/bin/env groovy
pipeline {
    agent any

    environment {
        appName = 'rest-demo'
        appVersion = '1.0.0'

        // aws cli credentials
        ACCESS_KEY = credentials('aws-dvo-access-key')
        SECRET_KEY = credentials('aws-dvo-secret-key')
        REGION = "ap-northeast-1"
    }

    stages {
        /*
         * Build Module (war file)
         */
        stage('Build Module') {
            steps {
                echo 'build module'
                echo ${ACCESS_KEY}
                //sh './sample/template/jenkins/run_maven_build.sh'
            }
        }

        /*
         * Build Container Image
         */
        stage('Build Image') {
            environment {
                BUILD_ARTIFACT = 'samples/template/target/rest-demo.war'
                IMAGE_NAME = 'rest-demo'
                IMAGE_TAG = 'v1'
            }
            steps {
                echo 'build image'
                //sh "cp ${BUILD_ARTIFACT} samples/template/container/src"
                //dir('samples/template/container/src') {
                //    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                //}
            }
        }

        /*
         * Register Container Image to ECR
         */ 
        stage('Register Image') {
            environment {
                ACCESS_TOKEN = credentials('aws-dvo-test-cicd-token')

            }
            steps {
                echo 'register image'
                // aws の設定を明示的にコマンドで渡す方法を調べる


                // aws login して token 取得？ or token を取得しておいて Jenkins に設定？

                //sh "docker login --username AWS --password ???? 572065744477.dkr.ecr.ap-northeast-1.amazonaws.com"
                //sh "docker tag rest-demo:latest 572065744477.dkr.ecr.ap-northeast-1.amazonaws.com/rest-demo:latest"
                //sh "docker push 572065744477.dkr.ecr.ap-northeast-1.amazonaws.com/rest-demo:latest"
                //sh "docker logout"
            }
        }
        /*
         * Rollout (update service)
         */ 
        stage('Rollout') {
            steps {
                echo 'rollout'
                // aws login しておいて credential をあらかじめ登録しておく？

                //sh "aws ecs update-service --cluster <cluster_name> --service <service_name> --force-new-deployment"
            }
        }
    }
}