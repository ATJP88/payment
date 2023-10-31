pipeline {

  environment {
    PROJECT = "intrepid-league-397203"
    APP_NAME = "pay"
    FE_SVC_NAME = "${APP_NAME}-pay"
    CLUSTER = "k8s-cluster-dev"
    CLUSTER_ZONE = "europe-west2-a"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}"
    JENKINS_CRED = "${PROJECT}"
  }

  agent {
    kubernetes {
      inheritFrom 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  # serviceAccountName: cd-jenkins
  containers:
  - name: gradle-bld
    image: openjdk:latest
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/google.com/cloudsdktool/cloud-sdk
    command:
    - cat
    tty: true
  - name: kubectl
    image: google/cloud-sdk
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('codebuild') {
      steps {
        container('gradle-bld') {
          sh """
             ls -a && pwd 
          """
        }
      }
    }
    stage('Build and push image with Container Builder') {
      steps {
        container('gcloud') {
          sh "gcloud config list"
          sh "gcloud config set account gke-admin@intrepid-league-397203.iam.gserviceaccount.com"
          sh "gcloud auth activate-service-account --key-file=./service-account.json"
          sh "gcloud config list"
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG} ."
        }
      }
    } 
    stage('Deploy Dev') {
      steps {
        container('kubectl') {
          
          sh "gcloud container clusters get-credentials hipstar --zone us-central1-c --project srinag"
          sh "kubectl apply -f paymentservice.yaml"
        }
      }
    }
  }
}