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
        stage('Code Analysis') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Code analysis stage started")
                }
                sh '''
                    sudo docker run --rm \
                    -e SONAR_HOST_URL="http://54.245.145.122:9000" \
                    -e SONAR_TOKEN="sqp_ba55720494d3b95c572b1182d6705cfaec2f34e4" \
                    -v "$PWD:/usr/src" \
                    sonarsource/sonar-scanner-cli \
                    -Dsonar.projectKey=lms
                '''
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Code analysis stage completed")
                }
            }
        }

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
                    env.BACKEND_IMAGE_TAG = "${BACKEND_IMAGE}:${env.VERSION}"
                    env.FRONTEND_IMAGE_TAG = "${FRONTEND_IMAGE}:${env.VERSION}"
                    slackSend(channel: SLACK_CHANNEL, message: "Read Version stage completed with version ${env.VERSION}")
                }
            }
        }

        stage('Build and Push Backend Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage started")
                }
                script {
                    docker.build("${BACKEND_IMAGE}:${env.VERSION}", 'api')
                    docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
                        docker.image("${BACKEND_IMAGE}:${env.VERSION}").push()
                    }
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage completed")
                }
            }
        }

        stage('Apply Backend Deployment') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage started")
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                        kubectl apply -f ${WORKSPACE}/api/be-deployment.yml
                        kubectl apply -f ${WORKSPACE}/api/be-service.yml
                    """
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage completed")
                }
            }
        }

        stage('Build and Push Frontend Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Frontend Image stage started")
                }
                script {
                    docker.build("${FRONTEND_IMAGE}:${env.VERSION}", 'webapp')
                    docker.withRegistry('', env.DOCKER_CREDENTIALS_ID) {
                        docker.image("${FRONTEND_IMAGE}:${env.VERSION}").push()
                    }
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Frontend Image stage completed")
                }
            }
        }

        stage('Apply Frontend Deployment') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Frontend Deployment stage started")
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                        kubectl apply -f ${WORKSPACE}/webapp/fe-deployment.yml
                        kubectl apply -f ${WORKSPACE}/webapp/fe-service.yml
                    """
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Frontend Deployment stage completed")
                }
            }
        }

        stage('Approval') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        slackSend(
                            channel: '#devops-projects', 
                            message: "Approval stage started for ${env.JOB_NAME} (<http://54.245.145.122:8080/job/lms-deployment-pipeline/${env.BUILD_NUMBER}/|Job Link>)", 
                            tokenCredentialId: "${SLACK_CREDENTIALS_ID}"
                        )
                        input message: 'Approve to Deploy', ok: 'Yes'
                    }
                    slackSend(
                        channel: '#devops-projects', 
                        color: '#439FE0', 
                        message: 'Request to build approved', 
                        tokenCredentialId: "${SLACK_CREDENTIALS_ID}"
                    )
                }
            }
        }

        stage('Notify After Approval') {
            steps {
                script {
                    slackSend(
                        channel: '#devops-projects', 
                        color: '#439FE0', 
                        message: 'LMS production deployment started', 
                        tokenCredentialId: "${SLACK_CREDENTIALS_ID}"
                    )
                }
            }
        }
    }

    post {
        always {
            script {
                def consoleOutput = ""
                try {
                    def logFile = currentBuild.rawBuild.getLog(1000) // Change 1000 to the number of lines you want to fetch
                    consoleOutput = logFile.join('\n')
                    slackSend(
                        channel: '#devops-projects',
                        color: '#439FE0',
                        message: "Build Console Output:\n```${consoleOutput}```",
                        tokenCredentialId: "${SLACK_CREDENTIALS_ID}"
                    )
                } catch (Exception e) {
                    println("Failed to read console output: ${e.message}")
                }
            }
        }
    }
}
