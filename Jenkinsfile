pipeline {
    agent any

    tools {
        nodejs 'NodeJS 24.1.0'
    }

    environment {
        CI = 'true'
        NODE_ENV = 'production'
        BUILD_DOCKER = 'true'
        DOCKER_IMAGE = 'unknowntw/my-nodejs-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('Practical5-6') { 
                    echo 'Installing dependencies...'
                    sh 'npm --version'
                    sh 'node --version'
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                dir('Practical5-6') {
                    sh 'ls -la node_modules/react-scripts || echo "react-scripts not found"'
                    sh 'npm test'
                }
            }
            post {
                always {
                    echo 'Test stage completed'
                }
                success {
                    echo 'All tests passed!'
                }
                failure {
                    echo 'Some tests failed!'
                }
            }
        }

        stage('Build Application') {
            steps {
                dir('Practical5-6') {
                    echo 'Building React application...'
                    sh 'npm run build'
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'Practical5-6/build/**/*', allowEmptyArchive: true
                }
            }
        }

        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'master'
                    environment name: 'BUILD_DOCKER', value: 'true'
                }
            }
            steps {
                echo 'Building Docker image...'
                script {
                    try {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                        echo "Docker image built successfully"
                    } catch (Exception e) {
                        echo "Docker build failed: ${e.getMessage()}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                anyOf {
                    branch 'master'
                    environment name: 'BUILD_DOCKER', value: 'true'
                }
            }
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'master') {
                        sh 'npm run deploy:prod'
                    } else {
                        sh 'npm run deploy:stage'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            script {
                if (env.BRANCH_NAME == 'master') {
                    echo "Docker image pushed: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
