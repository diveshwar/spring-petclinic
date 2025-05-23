pipeline {
    agent any
    environment {
        IMAGE = "diveshwar/petclinic:${BUILD_NUMBER}"
        COLOR = "green" // Change to "blue" manually or use logic to alternate
    }
    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/diveshwar/spring-petclinic.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE .'
            }
        }
        stage('Push Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u diveshwar --password-stdin'
                    sh 'docker push $IMAGE'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl set image deployment/petclinic-$COLOR petclinic=$IMAGE --record'
            }
        }
        stage('Switch Service') {
            steps {
                input "Switch traffic to $COLOR version?"
                sh '''
                kubectl patch service petclinic-service -p '{
                    "spec": {
                        "selector": {
                            "app": "petclinic",
                            "color": "'$COLOR'"
                        }
                    }
                }'
                '''
            }
        }
    }
}


































pipeline {
    agent any

    tools {
        // Make sure Maven is configured as 'Maven 3.8.1' in Jenkins Global Tool Configuration
        maven 'Maven 3.8.1'
    }

    environment {
        DOCKER_IMAGE = "diveshwar/spring-petclinic:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/diveshwar/spring-petclinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([url: '', credentialsId: 'docker-hub-creds']) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl set image deployment/spring-petclinic-deployment spring-petclinic-container=$DOCKER_IMAGE'
            }
        }
    }
}
