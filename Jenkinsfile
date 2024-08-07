pipeline {
    agent {
        label 'j-server'
    }

    environment {
        KUBECONFIG_CREDENTIAL_ID = 'k8s'
        version = "backend_${env.BUILD_NUMBER}"
        docker_image = "amzath0304/backend:${version}"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/AMZATH0320/backend.git'
            }
        }

       stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo \"$DOCKER_PASSWORD\" | sudo docker login --username \"$DOCKER_USERNAME\" --password-stdin"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerfilePath = '.'
                   sh "sudo docker build -t 'amzath0304/backend:${version}' ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    sh "sudo docker push 'amzath0304/backend:${version}'"
                }
            }
        } 
        stage('Security Scan') {
            steps {
                script {
                    def outputFilePath = "${env.WORKSPACE}/trivy_scan.txt"
                    sh "sudo trivy image ${docker_image} > ${outputFilePath}"
                    sh "cat ${outputFilePath}"
                }
            }
        }
        stage('Cleanup Docker Images') {
            steps {
                script {
                    sh "sudo docker rmi -f ${env.docker_image}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {

                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    script {
                        def kubeconfigPath = env.KUBECONFIG
                      withEnv(["VERSION=${env.version}"])
                      {
                         sh "echo ${VERSION}"
                         sh "export KUBECONFIG=${kubeconfigPath}"
                         sh "kubectl scale deploy backend-api --replicas=0 -n three-tier"
                         sh" sed -i 's/VERSION/${VERSION}/g' backend.yml"
                         sh " cat backend.yml"
                         sh "kubectl apply -f backend.yml --validate=false"
                         sh "kubectl get pods -n three-tier"
                         sh "kubectl get svc -n three-tier"
                         // haha
                      }
                    }
                }
            }
        }
    }
}
