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
        
        stage('Fix Dependencies (No sudo, No unzip dependency)') {
            steps {
                sh '''
                    set -e

                    echo "Checking AWS CLI..."

                    if ! command -v aws &> /dev/null; then
                        echo "Installing AWS CLI using pip (NO unzip needed)..."

                        # install AWS CLI v1 using Python (no unzip required)
                        python3 -m pip install --user awscli

                        export PATH=$PATH:$HOME/.local/bin
                    fi

                    aws --version

                    echo "Checking kubectl..."

                    if ! command -v kubectl &> /dev/null; then
                        echo "Installing kubectl..."

                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

                        chmod +x kubectl
                        mkdir -p $HOME/bin
                        mv kubectl $HOME/bin/

                        export PATH=$PATH:$HOME/bin
                    fi

                    kubectl version --client || true
                '''
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
