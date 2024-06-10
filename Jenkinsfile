pipeline {
    agent any

    environment {
        EKS_CLUSTER_NAME = 'lms-cluster'
        AWS_REGION = 'us-west-2'
        DOCKER_REGISTRY = 'awil360'
        FRONTEND_IMAGE = "${DOCKER_REGISTRY}/lms-front"
        BACKEND_IMAGE = "${DOCKER_REGISTRY}/lms-backend"
        DOCKER_CREDENTIALS_ID = 'lms-docker'
        SLACK_CHANNEL = '#devops-projects'
        SLACK_CREDENTIALS_ID = 'Secret text' // Replace with the actual credentials ID for your Slack token
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

        stage('Deploy Database and ConfigMap') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Deploy Database and ConfigMap stage started")
                    sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                    kubectl apply -f ${WORKSPACE}/api/pg-secret.yml
                    kubectl apply -f ${WORKSPACE}/api/pg-deployment.yml
                    kubectl apply -f ${WORKSPACE}/api/pg-service.yml
                    kubectl apply -f ${WORKSPACE}/api/be-configmap.yml
                    """
                    slackSend(channel: SLACK_CHANNEL, message: "Deploy Database and ConfigMap stage completed")
                }
            }
        }
    }
}

//         stage('Read Version') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Read Version stage started")
//                     def packageJson = readJSON file: 'webapp/package.json'
//                     env.VERSION = packageJson.version
//                     env.BACKEND_IMAGE_TAG = "${env.BACKEND_IMAGE}:${env.VERSION}"
//                     env.FRONTEND_IMAGE_TAG = "${env.FRONTEND_IMAGE}:${env.VERSION}"
//                     slackSend(channel: SLACK_CHANNEL, message: "Read Version stage completed with version ${env.VERSION}")
//                 }
//             }
//         }

//         stage('Build backend Docker Image') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Build backend Docker Image stage started")
//                     docker.build(env.BACKEND_IMAGE_TAG, 'api')
//                     slackSend(channel: SLACK_CHANNEL, message: "Build backend Docker Image stage completed with image tag ${env.VERSION}")
//                 }
//             }
//         }

//         stage('Push backend Docker Image') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Push backend Docker Image stage started")
//                     docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
//                         docker.image(env.BACKEND_IMAGE_TAG).push()
//                     }
//                     slackSend(channel: SLACK_CHANNEL, message: "Push backend Docker Image stage completed")
//                 }
//             }
//         }

//         stage('Build frontend Docker Image') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Build frontend Docker Image stage started")
//                     docker.build(env.FRONTEND_IMAGE_TAG, 'webapp')
//                     slackSend(channel: SLACK_CHANNEL, message: "Build frontend Docker Image stage completed with image tag ${env.VERSION}")
//                 }
//             }
//         }

//         stage('Push frontend Docker Image') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Push frontend Docker Image stage started")
//                     docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
//                         docker.image(env.FRONTEND_IMAGE_TAG).push()
//                     }
//                     slackSend(channel: SLACK_CHANNEL, message: "Push frontend Docker Image stage completed")
//                 }
//             }
//         }

//         stage('Deploy backend to EKS') {
//             steps {
//                 script {
//                     slackSend(channel: SLACK_CHANNEL, message: "Deploy backend to EKS stage started")
//                     sh """
//                     aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
//                     kubectl apply -f ${WORKSPACE}/api/be-deployment.yml
//                     kubectl apply -f ${WORKSPACE}/api/be-service.yml
//                     """
//                     slackSend(channel: SLACK_CHANNEL, message: "Deploy backend to EKS stage completed")
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
