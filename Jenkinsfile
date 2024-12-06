pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('1ede0dfb-20f1-46cb-9599-1dd484d9b50e')
        IMAGE_NAME = "geraldakenji/app-image:v-0.0${env.BUILD_NUMBER}-stable"
    }

    stages {
       stage("Clean Workspace") {
            steps {
                echo "Cleaning workspace..."
                cleanWs()
            }
       }
       stage("Git Checkout") {
            steps {
                echo "Checking out Git repository..."
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/GeraldAkenji/Ecommerce-Django.git']])
            }
       }
       stage("Install Dependencies") {
            steps {
                script {
                    echo "Setting up Python environment..."
                    sh "python3 --version"
                    sh "python3 -m venv venv"
                    sh ". venv/bin/activate"
                    sh "python3 -m pip install -r requirements.txt --no-cache-dir --break-system-packages"
                }
            }
        }
       stage("Docker Login") {
            steps {
                script {
                    echo "Logging into DockerHub..."
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                }
            }
       }
       stage("Build Docker Image") {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t $IMAGE_NAME ."
                }
            }
       }
       stage("Push to DockerHub") {
            steps {
                script {
                    echo "Pushing Docker image to DockerHub..."
                    sh "docker push $IMAGE_NAME"
                }
            }
       }
       stage("Deploy to Kubernetes") {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    dir('./k8s') {
                        withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: '88a9f11c-11e5-4bdb-b3bd-f63dba417648', namespace: 'gerald-env', serverUrl: '']]) {
                            sh "sed -i 's|PLEASE_REPLACE_ME|$IMAGE_NAME|g' deployment.yaml"
                            sh "kubectl apply -f ."
                        }
                    }
                }
            }
       }
    }
}
