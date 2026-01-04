pipeline {
    agent any

    tools {
        maven 'Maven-3'
    }

    stages {
        // Jenkins clones the code automatically, so no 'Git Clone' stage is needed!

        stage('Mvn Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                script {
                    sh 'docker build --no-cache -t jenkins-demo-svc .'
                    sh 'docker rm -f jenkins-demo-svc-container || true'
                    sh 'docker run -d --name jenkins-svc-demo-container -p 9090:8081 jenkins-demo-svc'
                }
            }
        }
    }
}
