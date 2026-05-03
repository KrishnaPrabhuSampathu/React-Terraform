pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {   
        AWS_REGION = 'us-east-1'
        CLUSTER_NAME = 'trend-eks-cluster'
        DOCKER_IMAGE = 'krishnaprabhu616/trend-app:latest'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        AWS_CREDENTIALS_ID = 'aws-creds'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', 
                credentialsId: 'github-creds',
                url: 'https://github.com/KrishnaPrabhuSampathu/Trend.git'
            }
        }
     

        stage('Update kubeconfig') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: AWS_CREDENTIALS_ID
                ]]) {
                    sh """
                        aws eks update-kubeconfig \
                          --region ${AWS_REGION} \
                          --name ${CLUSTER_NAME}
                    """
                }
            }
        }

        // stage('Install Dependencies') {
        //     steps {
        //         sh 'npm install'
        //     }
        // }

        // stage('Run Tests') {
        //     steps {
        //         sh 'npm test -- --watchAll=false'
        //     }
        // }

        // stage('Build React Application') {
        //     steps {
        //         sh 'npm run build'
        //     }
        // }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: DOCKER_CREDENTIALS_ID, url: '']) {
                    sh 'docker push ${DOCKER_IMAGE}'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
                sh 'kubectl rollout status deployment/trend-app'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl get svc'
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully to Amazon EKS.'
        }

        failure {
            echo 'Pipeline failed. Check logs for details.'
        }

        always {
            cleanWs()
        }
    }
}
