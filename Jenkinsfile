pipeline {
    environment {
        DOCKER_HOST = 'tcp://host.docker.internal:2375'
        CLOUDSDK_CONFIG = "${WORKSPACE}/.gcloud"
    }
    agent {
        docker {
            image 'radeczu/gcloud-sdk'
            args '--add-host=host.docker.internal:host-gateway -e DOCKER_HOST=tcp://host.docker.internal:2375'
        }
    }
    triggers {
        pollSCM '*/5 * * * *'
    }
    stages {

        stage('Prepare') {
            steps {
                sh '''
                    rm -f .env admin-token.txt
                    mkdir -p $CLOUDSDK_CONFIG
                '''
            }
        }

        stage('Authenticate') {
            steps {
                withCredentials([file(credentialsId: 'gcloud-sa-key', variable: 'GCP_KEY')]) {
                    echo "Authenticating with GCP..."
                    sh '''
                        gcloud auth activate-service-account --key-file=$GCP_KEY
                        gcloud config set project devopstraining-459716
                    '''
                }
            }
        }

        stage('Fetch Secrets') {
            steps {
                sh '''
                    echo "Fetching secrets from Google Secret Manager..."
                    echo K8S_SA_EMAIL=$(gcloud secrets versions access latest --secret="k8s-admin-sa-email") >> .env
                '''
            }
        }

        stage('K8S Deploy') {
            steps {
                sh '''
                    source .env

                    echo "Impersonating service account to get cluster credentials..."
                    gcloud auth print-access-token --impersonate-service-account=$K8S_SA_EMAIL > admin-token.txt

                    gcloud container clusters get-credentials gloud-k8s-cluster \
                        --region us-central1 \
                        --project devopstraining-459716 \
                        --access-token=$(cat admin-token.txt)

                    echo "Deploying Helm chart..."
                    helm install my-app ./charts/my-app
                '''
            }
        }
    }
}