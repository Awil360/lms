pipeline {
    agent any

    environment {
        EKS_CLUSTER_NAME = 'production-cluster'
        AWS_REGION = 'us-west-2'
        DOCKER_REGISTRY = 'awil360'
        DOCKER_IMAGE = 'awil360/lms'
        DOCKER_CREDENTIALS_ID = 'lms-docker'
        SLACK_CHANNEL = '#devops-projects'
        SLACK_CREDENTIALS_ID = 'Secret text'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Checkout stage started")
                }
                checkout scm
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Checkout stage completed")
                }
            }
        }
    

        stage('Read Version') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Read Version stage started")
                    def packageJson = readJSON file: 'webapp/package.json'
                    env.APP_VERSION = packageJson.version
                    slackSend(channel: SLACK_CHANNEL, message: "Read Version stage completed with version ${env.APP_VERSION}")
                }
            }
        }

        stage('Build backend Docker Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build Docker Image stage started")
                    docker.build("${env.DOCKER_IMAGE}:${env.APP_VERSION},'api'")
                    slackSend(channel: SLACK_CHANNEL, message: "Build Docker Image stage completed with image tag ${env.APP_VERSION}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Push Docker Image stage started")
                    docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
                        docker.image("${env.DOCKER_IMAGE}:${env.APP_VERSION}").push()
                    }
                    slackSend(channel: SLACK_CHANNEL, message: "Push Docker Image stage completed")
                }
            }
        }
    }
}

//         stage('Deploy to EKS') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Deploy to EKS stage started")
//                     sh """
//                     aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER_NAME
//                     kubectl set image deployment/my-app my-app=${env.DOCKER_IMAGE}:${env.APP_VERSION}
//                     """
//                     slackSend(channel: SLACK_CHANNEL, message: "Deploy to EKS stage completed")
//                 }
//             }
//         }

//         stage('Production Approval') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Waiting for production approval", color: '#FFFF00')
//                     input message: 'Approve deployment to production?', ok: 'Deploy'
//                     slackSend(channel: SLACK_CHANNEL, message: "Production deployment approved", color: '#00FF00')
//                 }
//             }
//         }
//     }

//     post {
//         success {
//             script {
//                 slackSend(channel: SLACK_CHANNEL, message: "Pipeline completed successfully", color: '#00FF00')
//             }
//         }

//         failure {
//             script {
//                 slackSend(channel: SLACK_CHANNEL, message: "Pipeline failed", color: '#FF0000')
//             }
//         }
//     }
// }
