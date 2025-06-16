pipeline {
    environment {
        DOCKER_HOST = 'tcp://host.docker.internal:2375'
        CLOUDSDK_CONFIG = "${WORKSPACE}/.gcloud"
        HOME = ${WORKSPACE}
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
                    mkdir -p $HOME/.kube
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

                    gcloud secrets versions access latest --secret="k8s_admin_access" > /tmp/gcp-key.json
                    gcloud auth activate-service-account --key-file=/tmp/gcp-key.json

                    gcloud config set project devopstraining-459716

                    gcloud container clusters get-credentials gloud-k8s-cluster \
                        --region us-central1 \
                        --project devopstraining-459716 \
                        --kubeconfig=$HOME/.kube/config

                    echo "Deploying Helm chart..."
                    helm install my-app ./charts/my-app
                '''
          }
        }
    }
}