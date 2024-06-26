pipeline { 
    environment {
        IMAGE = "jairodh/examen"
        LOGIN = "DOCKER_HUB"
    }
    agent none
    stages {
        stage('Build And Test CRUD') {
            agent any
            stages {
                stage('Clone') {
                    steps {
                        git branch: 'main', url: 'https://github.com/JairoDH/phpcrud.git'
                    }
                }
            }
        }
        stage('Build-Image') {
            agent any
            stages {
                stage('build-image') {
                    steps {
                        script { 
                            App = docker.build("${IMAGE}:${BUILD_NUMBER}")
                        }
                    }
                }
                stage('Up-images') {
                    steps {
                        script {
                            docker.withRegistry('', LOGIN) {
                                App.push()
                            }
                        }
                    }
                }
                stage('Remove-image') {
                    steps {
                        sh "docker rmi ${IMAGE}:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('Deployment') {
            agent any
            steps {
                script {
                    sshagent(credentials: ['VPS_SSH']) {
                        sh "ssh -o StrictHostKeyChecking=no jairo@fekir.touristmap.es wget https://raw.githubusercontent.com/JairoDH/phpcrud/main/docker-compose.yaml -O examen/docker-compose.yaml"
                        sh "ssh -o StrictHostKeyChecking=no jairo@fekir.touristmap.es cd examen/ && docker compose up -d --force-recreate"
                    }
                }
            }
        } 
    }
    post {
        always {
            mail to: 'jairodominguez3594@gmail.com',
            subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
    }
}

