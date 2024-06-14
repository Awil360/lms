pipeline {
    agent any

    environment {
        EKS_CLUSTER_NAME = 'lms-cluster'
        AWS_REGION = 'us-west-2'
        DOCKER_REGISTRY = 'awil360'
        FRONTEND_IMAGE = "${DOCKER_REGISTRY}/lms-fe"
        BACKEND_IMAGE = "${DOCKER_REGISTRY}/lms-be"
        DOCKER_CREDENTIALS_ID = 'lms-docker'
        SLACK_CHANNEL = '#devops-projects'
        SLACK_CREDENTIALS_ID = 'slack-token' // Replace with the actual credentials ID for your Slack token
        AWS_CREDENTIALS_ID = 'aws-credentials' // Replace with your actual AWS credentials ID
    }

    // stages {
    //     stage('Code Analysis') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Code analysis stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //             sh '''
    //                 sudo docker run --rm \
    //                 -e SONAR_HOST_URL="http://54.200.210.163:9000" \
    //                 -e SONAR_TOKEN="sqp_ba55720494d3b95c572b1182d6705cfaec2f34e4" \
    //                 -v "$PWD:/usr/src" \
    //                 sonarsource/sonar-scanner-cli \
    //                 -Dsonar.projectKey=lms
    //             '''
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Code analysis stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

    //     stage('Checkout') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Checkout stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //             checkout scm
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Checkout stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

    //     stage('Deploy Database and ConfigMap') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Deploy Database and ConfigMap stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //             withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
    //                 sh """
    //                     aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
    //                     kubectl apply -f ${WORKSPACE}/api/pg-secret.yml
    //                     kubectl apply -f ${WORKSPACE}/api/pg-deployment.yml
    //                     kubectl apply -f ${WORKSPACE}/api/pg-service.yml
    //                     kubectl apply -f ${WORKSPACE}/api/be-configmap.yml
    //                 """
    //             }
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Deploy Database and ConfigMap stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

    //     stage('Read Version') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Read Version stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //                 def packageJson = readJSON file: 'webapp/package.json'
    //                 env.VERSION = packageJson.version
    //                 env.BACKEND_IMAGE_TAG = "${BACKEND_IMAGE}:${env.VERSION}"
    //                 env.FRONTEND_IMAGE_TAG = "${FRONTEND_IMAGE}:${env.VERSION}"
    //                 slackSend(channel: SLACK_CHANNEL, message: "Read Version stage completed with version ${env.VERSION}", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

    //     stage('Build and Push Backend Image') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //             script {
    //                 docker.build("${BACKEND_IMAGE}:${env.VERSION}", 'api')
    //                 docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
    //                     docker.image("${BACKEND_IMAGE}:${env.VERSION}").push()
    //                 }
    //             }
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

    //     stage('Apply Backend Deployment') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //             withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
    //                 sh """
    //                     aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
    //                     kubectl apply -f ${WORKSPACE}/api/be-deployment.yml
    //                     kubectl apply -f ${WORKSPACE}/api/be-service.yml
    //                 """
    //             }
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

    //     stage('Build and Push Frontend Image') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Build and Push Frontend Image stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //             script {
    //                 docker.build("${FRONTEND_IMAGE}:${env.VERSION}", 'webapp')
    //                 docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
    //                     docker.image("${FRONTEND_IMAGE}:${env.VERSION}").push()
    //                 }
    //             }
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Build and Push Frontend Image stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

    //     stage('Apply Frontend Deployment') {
    //         steps {
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Apply Frontend Deployment stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //             withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
    //                 sh """
    //                     aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
    //                     kubectl apply -f ${WORKSPACE}/webapp/fe-deployment.yml
    //                     kubectl apply -f ${WORKSPACE}/webapp/fe-service.yml
    //                 """
    //             }
    //             script {
    //                 slackSend(channel: SLACK_CHANNEL, message: "Apply Frontend Deployment stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
    //             }
    //         }
    //     }

        stage('Approval') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        slackSend(
                            channel: env.SLACK_CHANNEL, 
                            message: "Approval stage started for ${env.JOB_NAME} (<http://54.200.210.163:8080/job/lms-deployment-pipeline/|Job Link>)", 
                            tokenCredentialId: env.SLACK_CREDENTIALS_ID
                        )
                        input message: 'Approve to Deploy', ok: 'Yes'
                    }
                    slackSend(
                        channel: env.SLACK_CHANNEL, 
                        color: '#439FE0', 
                        message: 'Request to build approved', 
                        tokenCredentialId: env.SLACK_CREDENTIALS_ID
                    )
                }
            }
        }

        stage('Notify After Approval') {
            steps {
                script {
                    slackSend(
                        channel: env.SLACK_CHANNEL, 
                        color: '#439FE0', 
                        message: 'LMS production deployment started', 
                        tokenCredentialId: env.SLACK_CREDENTIALS_ID
                    )
                }
            }
        }
    }

  post {
    always {
        script {
            def buildLogPath = "${WORKSPACE}/build.log"
            if (fileExists(buildLogPath)) {
                try {
                    def buildLogContent = readFile(buildLogPath).trim()
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: '#439FE0',
                        message: "```${env.JOB_NAME}```\nBuild Console Output:\n```\n${buildLogContent}\n```",
                        tokenCredentialId: env.SLACK_CREDENTIALS_ID
                    )
                } catch (Exception e) {
                    println("Failed to send build log to Slack: ${e.message}")
                    slackSend(
                        channel: env.SLACK_CHANNEL,
                        color: '#FF0000',
                        message: "```${env.JOB_NAME}```\nFailed to send build log to Slack: ${e.message}",
                        tokenCredentialId: env.SLACK_CREDENTIALS_ID
                    )
                }
            } else {
                println("Build log file (${buildLogPath}) not found.")
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: '#FF0000',
                    message: "```${env.JOB_NAME}```\nFailed to capture build log: Log file not found.",
                    tokenCredentialId: env.SLACK_CREDENTIALS_ID
                )
            }
        }
    }
}



