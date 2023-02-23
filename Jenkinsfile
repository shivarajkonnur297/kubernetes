pipeline {
    agent any

    stages {
        stage('Clone Repo') {
            steps {
                sh 'rm -rf shivarajkonnur297/kubernetes'
                sh 'git clone shivarajkonnur297.kubernetes'
            }
        }
         
        stage('Install_Docker') {
            steps {
                script {
                    def dockerPath = sh(script: 'which docker', returnStatus: true)
                    if (dockerPath != 0) {
                        sh 'apt update'
                        sh 'apt-get install docker.io -y -q'
                    } else {
                        echo 'Docker is already installed'
                  }
               }
            }
         }        

        stage('Build Image') {
            steps {
                sh 'docker build -t your-dockerhub-username/node-app:latest .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    #!/bin/bash
                    docker stop my_project-1 || true
                    docker rm my_project-1 || true
                    docker run -d -p 8081:80 --name my_project-1 your-dockerhub-username/node-app:latest
                '''
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-cred', variable: 'dockerhub-cred')]) {
                    sh '''
                        #!/bin/bash
                        docker login -u <Dockerhub_username> -p ${dockerhub-cred}
                        docker push your-dockerhub-username/node-app:latest
                    '''
                }
            }
        }

        stage('Deploy to K8s Server') {
            agent {
                label 'k_master'
            }
            steps {
                script {
                    try {
                        sh 'kubectl apply -f deployment.yaml --record=true'
                        sh 'kubectl apply -f services.yaml'
                    } catch (error) {
                        sh 'kubectl create -f deployment.yaml'
                        sh 'kubectl create -f services.yaml'
                    }
                }
            }
        }
    }
}
