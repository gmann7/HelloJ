pipeline {
    agent any
    environment {
        GKE_PROJECT = credentials('gcp-project-id')  
        GKE_CLUSTER = credentials('gcp-cluster-name')  
        GKE_ZONE = 'us-central1'  
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-credentials-id') 
        GITHUB_TOKEN = credentials('github-token')  
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: "https://github.com/gmann7/HelloJ.git"
            }
        }
        stage('Set up GCloud') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gcp-credentials-id', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh '''
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project $GKE_PROJECT
                        gcloud config set compute/zone $GKE_ZONE
                        gcloud container clusters get-credentials $GKE_CLUSTER
                        '''
                    }
                }
            }
        }
        stage('Create Namespace') {
            steps {
                script {
                    def namespace = "review-${env.BRANCH_NAME}"
                    sh "kubectl create namespace ${namespace}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def namespace = "review-${env.BRANCH_NAME}"
                    sh """
                    kubectl apply -f k8s/deployment.yaml -n ${namespace}
                    kubectl apply -f k8s/service.yaml -n ${namespace}
                    """
                }
            }
        }
        stage('Comment PR with URL') {
            steps {
                script {
                    def namespace = "review-${env.BRANCH_NAME}"
                    def prUrl = "http://${namespace}.my-app.com"
                    sh """
                    curl -s -H "Authorization: token ${GITHUB_TOKEN}" \\
                      -X POST \\
                      -d "{\\"body\\": \\"Review app is available at: ${prUrl}\\"}" \\
                      "https://api.github.com/repos/your-repo/issues/${env.CHANGE_ID}/comments"
                    """
                }
            }
        }
        
    }
    post {
        changed {
            script {
                // Check if the current branch is merged into master
                if (env.BRANCH_NAME == 'master') {
                    // Destroy the previously deployed pod
                    sh "kubectl delete pod -n review-${env.BRANCH_NAME}"
                }
            }
        }
    }
}
