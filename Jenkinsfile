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

    stages {
        stage('Code Analysis') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Code analysis stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
                script {
                    def codeAnalysisOutput = sh(script: '''
                        sudo docker run --rm \
                        -e SONAR_HOST_URL="http://54.245.145.122:9000" \
                        -e SONAR_TOKEN="sqp_ba55720494d3b95c572b1182d6705cfaec2f34e4" \
                        -v "$PWD:/usr/src" \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=lms
                    ''', returnStdout: true).trim()
                    echo codeAnalysisOutput
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Code analysis stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

        stage('Checkout') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Checkout stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
                checkout scm
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Checkout stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

        stage('Deploy Database and ConfigMap') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Deploy Database and ConfigMap stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    script {
                        def deployOutput = sh(script: """
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                            kubectl apply -f ${WORKSPACE}/api/pg-secret.yml
                            kubectl apply -f ${WORKSPACE}/api/pg-deployment.yml
                            kubectl apply -f ${WORKSPACE}/api/pg-service.yml
                            kubectl apply -f ${WORKSPACE}/api/be-configmap.yml
                        """, returnStdout: true).trim()
                        echo deployOutput
                    }
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Deploy Database and ConfigMap stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

        stage('Read Version') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Read Version stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                    def packageJson = readJSON file: 'webapp/package.json'
                    env.VERSION = packageJson.version
                    env.BACKEND_IMAGE_TAG = "${BACKEND_IMAGE}:${env.VERSION}"
                    env.FRONTEND_IMAGE_TAG = "${FRONTEND_IMAGE}:${env.VERSION}"
                    slackSend(channel: SLACK_CHANNEL, message: "Read Version stage completed with version ${env.VERSION}", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

        stage('Build and Push Backend Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
                script {
                    def buildPushBackendOutput = sh(script: """
                        docker build -t ${BACKEND_IMAGE}:${env.VERSION} api
                        docker login -u <username> -p <password>
                        docker push ${BACKEND_IMAGE}:${env.VERSION}
                    """, returnStdout: true).trim()
                    echo buildPushBackendOutput
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Backend Image stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

        stage('Apply Backend Deployment') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    script {
                        def applyBackendOutput = sh(script: """
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                            kubectl apply -f ${WORKSPACE}/api/be-deployment.yml
                            kubectl apply -f ${WORKSPACE}/api/be-service.yml
                        """, returnStdout: true).trim()
                        echo applyBackendOutput
                    }
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Backend Deployment stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

        stage('Build and Push Frontend Image') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Frontend Image stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
                script {
                    def buildPushFrontendOutput = sh(script: """
                        docker build -t ${FRONTEND_IMAGE}:${env.VERSION} webapp
                        docker login -u <username> -p <password>
                        docker push ${FRONTEND_IMAGE}:${env.VERSION}
                    """, returnStdout: true).trim()
                    echo buildPushFrontendOutput
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Build and Push Frontend Image stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

        stage('Apply Frontend Deployment') {
            steps {
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Frontend Deployment stage started", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    script {
                        def applyFrontendOutput = sh(script: """
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                            kubectl apply -f ${WORKSPACE}/webapp/fe-deployment.yml
                            kubectl apply -f ${WORKSPACE}/webapp/fe-service.yml
                        """, returnStdout: true).trim()
                        echo applyFrontendOutput
                    }
                }
                script {
                    slackSend(channel: SLACK_CHANNEL, message: "Apply Frontend Deployment stage completed", tokenCredentialId: SLACK_CREDENTIALS_ID)
                }
            }
        }

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
                def consoleOutput = currentBuild.rawBuild.getLog(1000).join("\n")
                slackSend(
                    channel: env.SLACK_CHANNEL,
                    color: '#439FE0',
                    message: "Build Console Output:\n```${consoleOutput}```",
                    tokenCredentialId: env.SLACK_CREDENTIALS_ID
                )
            }
        }
    }
}
