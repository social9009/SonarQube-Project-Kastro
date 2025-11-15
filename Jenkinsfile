pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_USERNAME = 'hitfast'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/spotify-app:latest"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KastroVKiran/SonarQube-Project-Kastro.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Kastro \
                        -Dsonar.projectKey=KastroKey \
                        -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn package"
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Docker Push to DockerHub') {
            steps {
                withCredentials([string(credentialsId: 'hitfast', variable: 'DOCKERHUB_TOKEN')]) {
                    sh """
                    echo \$DOCKERHUB_TOKEN | docker login -u ${DOCKERHUB_USERNAME} --password-stdin
                    docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Run Docker Container') {
            steps {
        script {
            withCredentials([usernamePassword(
                credentialsId: 'hitfast',
                usernameVariable: 'DOCKERHUB_USERNAME',
                passwordVariable: 'DOCKERHUB_PASSWORD'
            )]) {
                sh """
                    echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                    docker push hitfast/spotify-app:latest
                """
               }
           }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}




