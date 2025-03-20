pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = credentials('aws-credentials')
        KUBECONFIG = credentials('kubeconfig')
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_PASS')]) {
                        sh """
                        echo "Cloning GitHub repository..."
                        git clone https://$GITHUB_USER:$GITHUB_PASS@github.com/yogesh-koli/techneai-devops-code.git
                        cd techneai-devops-code
                        git checkout master
                        """
                    }
                }
            }
        }

        stage('Build and Push Docker Images') {
            steps {
                script {
                    def images = ['Alignment Detection', 'Element Detection', 'Font Detection', 'Label&text', 'Label Sequence', 'spell check']

                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        images.each { image ->
                            def imageName = image.replaceAll(' ', '-').replaceAll('&', 'and').toLowerCase()

                            sh """
                            echo "Logging into DockerHub..."
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                            echo "Building Docker Image: dockerhubuser/${imageName}:latest"
                            docker build -t dockerhubuser/${imageName}:latest ./${imageName}

                            echo "Pushing Image to DockerHub..."
                            docker push dockerhubuser/${imageName}:latest
                            """
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                        sh '''
                        echo "Deploying to Kubernetes..."
                        kubectl apply -f uiux-deployment.yaml
                        kubectl apply -f uiux-service.yaml
                        kubectl apply -f uiux-ingress.yaml
                        kubectl apply -f uiux-ingress-resource.yaml
                        '''
                    }
                }
            }
        }
    }
}
