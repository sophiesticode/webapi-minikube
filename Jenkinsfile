pipeline {
    agent {
        label 'jenkins-slave'
    }
    environment {
        DOCKER_IMAGE = 'webapi:latest'
        GIT_URL = 'git@github-sophiesticode:sophiesticode/webapi-minikube.git'
        KUBERNETES_NAMESPACE_DEV = 'devops-project-dev'
	KUBERNETES_NAMESPACE_PROD = 'devops-project-prod'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-ssh', url: "$GIT_URL"
            }
        }
        stage('Build Docker Image') {
            steps {
		// with docker build
		// sh 'cd ./webapi/ && docker build -t $DOCKER_IMAGE -f Dockerfile .'
		// with pack
		sh 'cd ./webapi/ && pack build $DOCKER_IMAGE --builder heroku/builder:24'
            }
        }
        stage('Test Docker Image') {
            steps {
                sh 'docker run -d -p 8081:8081 $DOCKER_IMAGE'
                sh 'sleep 5'
                sh 'curl http://localhost:8081/whoami'
            }
        }
	stage('Stop Docker Container') {
	    steps {
		sh 'docker stop $(docker ps -q --filter ancestor=$DOCKER_IMAGE)'
		sh 'docker rm $(docker ps -a -q --filter ancestor=$DOCKER_IMAGE)'
            }
	}
	stage('Load Image to Minikube') {
            steps {
                sh 'docker save $DOCKER_IMAGE -o app.tar && minikube image load app.tar'
            }
        }
	stage('Create Development Namespace if not exists') {
            steps {
                script {
                    def namespaceDevExists = sh (script: "kubectl get namespaces | grep $KUBERNETES_NAMESPACE_DEV", returnStatus: true)
                    if (namespaceDevExists != 0) {
                        sh 'kubectl create namespace $KUBERNETES_NAMESPACE_DEV'
                    }
                }
            }
        }
        stage('Deploy to Development') {
            steps {
                sh 'kubectl apply -f deployments/deployment-dev.yaml'
            }
        }
        stage('Test Development Deployment') {
            steps {
                sh 'kubectl wait --for=condition=available --timeout=60s deployment/webapi-deployment-dev --namespace=$KUBERNETES_NAMESPACE_DEV' 
                sh 'kubectl get pods --namespace=$KUBERNETES_NAMESPACE_DEV' 
                sh 'curl $(minikube service webapi-service-dev --url --namespace=$KUBERNETES_NAMESPACE_DEV)/whoami'
            }
        }
	stage('Create Production Namespace if not exists') {
            steps {
                script {
                    def namespaceProdExists = sh (script: "kubectl get namespaces | grep $KUBERNETES_NAMESPACE_PROD", returnStatus: true)
                    if (namespaceProdExists != 0) {
                        sh 'kubectl create namespace $KUBERNETES_NAMESPACE_PROD'
                    }
                }
            }
        }
        stage('Deploy to Production') {
            steps {
                sh 'kubectl apply -f deployments/deployment-prod.yaml'
            }
        }
	stage('Test Production Deployment') {
            steps {
		sh 'kubectl wait --for=condition=available --timeout=60s deployment/webapi-deployment-prod --namespace=$KUBERNETES_NAMESPACE_PROD'
                sh 'kubectl port-forward service/webapi-service-prod 8081:8081 --namespace=$KUBERNETES_NAMESPACE_PROD &'
                sh 'curl http://localhost:8081/whoami'
            }
        }
    }
}
