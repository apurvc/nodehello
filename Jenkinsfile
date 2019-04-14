def project = 'lbg-p-ob-e-playpen'
def  appName = 'nodehello'
def  feSvcName = "nodehello-svc"
def  imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

pipeline {
  agent {
    kubernetes {
      label 'default'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: jenkins-cd
  containers:
  - name: docker
    image: gcr.io/cloud-builders/docker
    command:
    - cat
    tty: true  
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }
  stages {
    stage('Build Release') {
      //when {
      //  branch 'master'
      //}
      steps {
        echo 'Image tag is ${imageTag} branch is ${env.BRANCH_NAME} and ${env.BUILD_NUMBER}'
        container('docker') {
        //git 'https://github.com/apurvc/nodehello.git'

        sh("docker build -t ${imageTag} .")
        sh("gcloud docker -- push ${imageTag}")
        }
      }
    }
/*    stage('Deploy Canary') {
     // Canary branch
      when { branch 'canary' }
      steps {
        container('kubectl') {
          // Change deployed image in canary to the one we just built
          sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/canary/*.yaml")
          sh("kubectl --namespace=production apply -f k8s/services/")
          sh("kubectl --namespace=production apply -f k8s/canary/")
          sh("echo http://`kubectl --namespace=production get service/${feSvcName} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcName}")
        } 
      }
    }
    stage('Deploy Production') {
      // Production branch
      when { branch 'master' }
      steps{
        container('kubectl') {
        // Change deployed image in canary to the one we just built
          sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/production/*.yaml")
          sh("kubectl --namespace=production apply -f k8s/services/")
          sh("kubectl --namespace=production apply -f k8s/production/")
          sh("echo http://`kubectl --namespace=production get service/${feSvcName} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcName}")
        }
      }
    }
*/    
    stage('Deploy Dev') {
      // Developer Branches
      when { 
        not { branch 'master' } 
        not { branch 'canary' }
      } 
      steps {
        
          echo 'Image tag is ${imageTag} branch is ${env.BRANCH_NAME} and ${env.BUILD_NUMBER}'

          // Create namespace if it doesn't exist
          sh("kubectl get ns ${env.BRANCH_NAME} || kubectl create ns ${env.BRANCH_NAME}")
          sh("kubectl --namespace=${env.BRANCH_NAME} apply -f deployment.yaml")
          echo 'To access your environment run `kubectl proxy`'
          sh("echo http://`kubectl --namespace=${env.BRANCH_NAME} get service/${feSvcName} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${feSvcName}")
        
      }     
    }
  }
}
