pipeline {
    agent any
    tools {
        maven 'maven_3.9.6_kavita' 
    }
    environment {
        DATE = new Date().format('yy.M')
        TAG = "${DATE}.${BUILD_NUMBER}"
    }
    stages {
        stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
         stage('SonarQube Analysis') {
            steps {
                // Use withSonarQubeEnv block to manage SonarQube environment
                withSonarQubeEnv('ramasonar') {
                    // Execute SonarQube analysis
                    sh '''
                      mvn clean compile sonar:sonar
                    '''
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    docker.build("rama1/hello-world:${TAG}")
                }
            }
        }
	    stage('Pushing Docker Image to Dockerhub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_credential') {
                        docker.image("rama1/hello-world:${TAG}").push()
                        docker.image("rama1/hello-world:${TAG}").push("latest")
                    }
                }
            }
        }
        stage('Deploy'){
            steps {
                sh "docker stop hello-world | true"
                sh "docker rm hello-world | true"
                sh "docker run --name hello-world -d -p 8080:8080 rama1/hello-world:${TAG}"
            }
        }
    }
}
