pipeline {
    agent any

    environment {
        IMAGE_NAME = "marammanai/discovery-service:latest"
        K8S_MASTER = "ceph1@192.168.13.11"
        DEPLOY_YAML = "k8s-discovery-deployment.yaml"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'discovery', url: 'https://github.com/Maram-web/discovery.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Copy YAML') {
            steps {
                sh '''
                    ssh-keyscan -H 192.168.13.11 >> ~/.ssh/known_hosts
                    scp $DEPLOY_YAML $K8S_MASTER:/home/ceph1/$DEPLOY_YAML
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh 'ssh $K8S_MASTER kubectl apply -f /home/ceph1/$DEPLOY_YAML'
            }
        }
    }

    post {
        success { echo "✅ discovery-service deployed!" }
        failure { echo "❌ discovery-service failed!" }
    }
}
