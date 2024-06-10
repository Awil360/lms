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
        SLACK_CREDENTIALS_ID = 'Secret text' // Replace with the actual credentials ID for your Slack token
        AWS_CREDENTIALS_ID = 'aws-credentials' // Replace with your actual AWS credentials ID
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
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                    kubectl apply -f ${WORKSPACE}/api/pg-secret.yml
                    kubectl apply -f ${WORKSPACE}/api/pg-deployment.yml
                    kubectl apply -f ${WORKSPACE}/api/pg-service.yml
                    kubectl apply -f ${WORKSPACE}/api/be-configmap.yml
                    """
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Deploy Database and ConfigMap stage completed")
                }
            }
        }

        stage('Read Version') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Read Version stage started")
                    def packageJson = readJSON file: 'webapp/package.json'
                    env.VERSION = packageJson.version
                    env.BACKEND_IMAGE_TAG = "${env.BACKEND_IMAGE}:${env.VERSION}"
                    env.FRONTEND_IMAGE_TAG = "${env.FRONTEND_IMAGE}:${env.VERSION}"
                    slackSend(channel: SLACK_CHANNEL, message: "Read Version stage completed with version ${env.VERSION}")
                }
            }
        }

        stage('Build and Push Backend Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage started")
                    docker.build("${BACKEND_IMAGE}:${env.VERSION}", 'api')
                    docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
                        docker.image("${BACKEND_IMAGE}:${env.VERSION}").push()
                    }
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage completed")
                }
            }
        }

        stage('Apply Backend Deployment') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage started")
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                        sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                        kubectl apply -f ${WORKSPACE}/api/be-deployment.yml
                        kubectl apply -f ${WORKSPACE}/api/be-service.yml
                        """
                    }
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage completed")
                }
            }
        }

        stage('Update Frontend .env') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Update Frontend .env stage started")
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    script {
                        def lbUrl = sh(
                            script: """
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                            kubectl get svc lms-be-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
                            """,
                            returnStdout: true
                        ).trim()

                        def envFile = readFile 'webapp/.env'
                        envFile = envFile.replaceAll(/REACT_APP_BACKEND_URL=.*/, "REACT_APP_BACKEND_URL=http://${lbUrl}:3000")
                        writeFile file: 'webapp/.env', text: envFile
                    }
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Update Frontend .env stage completed")
                }
            }
        }

        stage('Build and Push frontend Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push frontend Image stage started")
                    docker.build("${FRONTEND_IMAGE_TAG}:${env.VERSION}", 'webapp')
                    docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
                        docker.image("${FRONTEND_IMAGE}:${env.VERSION}").push()
                    }
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push frontend Image stage completed")
                }
            }
        }

        stage('Apply frontend Deployment') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply frontend Deployment stage started")
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                        sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                        kubectl apply -f ${WORKSPACE}/webapp/fe-deployment.yml
                        kubectl apply -f ${WORKSPACE}/webapp/fe-service.yml
                        """
                    }
                    slackSend(channel: SLACK_CHANNEL, message: "Apply frontend Deployment stage completed")
                }
            }
        }

        stage('Production Approval') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Waiting for production approval", color: '#FFFF00')
                    input message: 'Approve deployment to production?', ok: 'Deploy'
                    slackSend(channel: SLACK_CHANNEL, message: "Production deployment approved", color: '#00FF00')
                }
            }
        }
    }

    post {
        success {
            script {
                slackSend(channel: SLACK_CHANNEL, message: "Pipeline completed successfully", color: '#00FF00')
            }
        }

        failure {
            script {
                slackSend(channel: SLACK_CHANNEL, message: "Pipeline failed", color: '#FF0000')
            }
        }
    }
}
