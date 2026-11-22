
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building Spring Boot application...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t puja81/cicd-demo:${BUILD_NUMBER} .
                    docker tag puja81/cicd-demo:${BUILD_NUMBER} puja81/cicd-demo:latest
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push puja81/cicd-demo:${BUILD_NUMBER}
                        docker push puja81/cicd-demo:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application on EC2...'
                sh '''
                    docker pull puja81/cicd-demo:latest
                    docker stop cicd-demo || true
                    docker rm cicd-demo || true
                    docker run -d --name cicd-demo --restart unless-stopped -p 80:8080 puja81/cicd-demo:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed successfully!'
        }
        failure {
            echo 'FAILURE: Pipeline failed. Check Console Output.'
        }
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
