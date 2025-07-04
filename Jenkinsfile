pipeline {
    agent any
    environment {
        CI = 'true'
        // Use Jenkins credentials plugin references by ID
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id') // replace with your Jenkins credential ID
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key') // replace with your Jenkins credential ID
        AWS_DEFAULT_REGION = 'ap-southeast-2' // !! IMPORTANT: Replace with your actual AWS region !!

        AWS_S3_BUCKET = 'haiqal313-react-app' // Your S3 bucket for artifacts
        NODE_OPTIONS = '--openssl-legacy-provider' // Fix for Node 17+ crypto error
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:lts-buster-slim'
                    args '-p 3000:3000 --add-host=host.docker.internal:host-gateway'
                }
            }
            steps {
                echo 'Installing Node.js and npm...'
                // Commands to install Node.js and npm on Debian-based system
                echo 'Building React application...'
                sh 'npm install'
                sh 'npm run build'
                echo 'React build complete. Artifacts are in the "build/" directory.'
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:lts-buster-slim'
                    args '-p 3000:3000 --add-host=host.docker.internal:host-gateway'
                }
            }
            steps {
                echo 'Running tests...'
                sh 'npm test'
                echo 'Tests complete.'
            }
        }
        stage('Manual Approval') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        input message: 'Approve deployment to EC2?', ok: 'Deploy'
                    }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'xueshanf/awscli:alpine-3.16' // for AWS CLI + python pip support
                    args '-p 3001:3001 --add-host=host.docker.internal:host-gateway'
                    args '-u root' // Run as root to install packages
                }
            }
            steps {
                echo 'Initiating deployment to EC2 via S3 and CodeDeploy...'
                script {
                    echo "Uploading deployment_package.zip to S3 bucket: ${env.AWS_S3_BUCKET}..."
                    sh "aws s3 sync --acl public-read build s3://${env.AWS_S3_BUCKET}/"

                    echo 'Deployment package uploaded to S3.'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
