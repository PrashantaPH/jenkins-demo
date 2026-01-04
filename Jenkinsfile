pipeline {
    agent any

    tools {
        maven 'Maven-3'
    }

    // Define variables at the top to avoid typos in multiple stages
    environment {
        IMAGE_NAME = "jenkins-demo-svc"
        CONTAINER_NAME = "jenkins-demo-svc-container"
        HOST_PORT = "9091"
        APP_PORT = "8081"
    }

    stages {
        // Stage 1: Compile and package the Java JAR
        stage('Mvn Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // Stage 2: Build the Docker Image
        stage('Docker Image Build') {
            steps {
                // --no-cache ensures that code changes are picked up every time
                sh "docker build --no-cache -t ${IMAGE_NAME} ."
            }
        }

        // Stage 3: Clean up old container and Run the new one
        stage('Docker Deploy') {
            steps {
                script {
                    // 1. Remove the old container if it exists (using the exact same name)
                    sh "docker rm -f ${CONTAINER_NAME} || true"

                    // 2. Start the new container
                    sh "docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:${APP_PORT} ${IMAGE_NAME}"
                }
            }
        }
    }

    // Optional: Cleanup unused images to save disk space
    post {
        success {
            echo "Successfully deployed to http://localhost:${HOST_PORT}/api/message"
        }
        always {
            // Removes dangling images left over from the build process
            sh 'docker image prune -f'
        }
    }
}
