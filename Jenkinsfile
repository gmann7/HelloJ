pipeline {
    agent any
    environment {
        GKE_PROJECT = 'your-gcp-project-id'  // Sostituisci con il tuo ID progetto GCP
        GKE_CLUSTER = 'your-gke-cluster-name'  // Sostituisci con il nome del tuo cluster GKE
        GKE_ZONE = 'your-gke-cluster-zone'  // Sostituisci con la zona del tuo cluster GKE
        GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-credentials-id')  // ID delle credenziali GCP in Jenkins
        GITHUB_TOKEN = credentials('github-token')  // ID del token GitHub in Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/gmann7/HelloJ.git', credentialsId: GITHUB_TOKEN  
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
                    def namespace = "review-${env.CHANGE_ID}"
                    sh "kubectl create namespace ${namespace}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def namespace = "review-${env.CHANGE_ID}"
                    sh """
                    kubectl apply -f k8s/deployment.yaml -n ${namespace}
                    kubectl apply -f k8s/service.yaml -n ${namespace}
                    """
                }
            }
        }
        
    }
    post {
        always {
            script {
                def namespace = "review-${env.CHANGE_ID}"
                sh "kubectl delete namespace ${namespace}"
            }
        }
    }
}
