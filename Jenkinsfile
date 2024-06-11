pipeline { 
    environment {
        IMAGE = "jairodh/django"
        LOGIN = "DOCKER_HUB"
    }
    agent none
    stages {
        stage('Build And Test Django') {
            agent {
                docker { image 'python:3'
                args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'master',url:'https://github.com/JairoDH/phpcrud.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r requirements.txt'
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
                           App = docker.build "$IMAGE:$BUILD_NUMBER"
                      }
                   }
                }
                stage('Up-images') {
		   steps {
		       script {
			   docker.withRegistry( '', LOGIN) {
				App.push()
			   }
		       }
	           }
	        }  
                stage('Remove-image') {
		   steps {
                        sh "docker rmi $IMAGE:$BUILD_NUMBER"
                   }
                }
             }
	}
        stage('Deployment') {
	    agent any
            steps {
		script {
                     sshagent(credentials: ['VPS_SSH']) {
                         sh "ssh -o StrictHostKeyChecking=no jairo@fekir.touristmap.es wget https://raw.githubusercontent.com/JairoDH/phpcrud/master/docker-compose.yaml -O docker-compose.yaml"
                         sh "ssh -o StrictHostKeyChecking=no jairo@fekir.touristmap.es docker compose up -d --force-recreate"
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
