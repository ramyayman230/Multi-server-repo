pipeline {
    agent any 
    tools {
        maven 'Maven'
    }
    environment {
        dockerhub_cred = credentials('docker-cred')
        KUBECONFIG_CRED = credentials('kubeconfig')
    }
    
    stages {
        
        stage('maven build') {
            steps {
               sh 'mvn package'
            }
        }
        stage('docker build') {
            steps {
               sh 'docker build -t romeo23/server-${BRANCH_NAME}-jenkins:${BUILD_NUMBER} .'
            }
        }
        stage('list images') {
            steps {
               sh 'docker images'
            }
        }
        stage('dockerhub push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                        sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                        sh 'docker push romeo23/server-${BRANCH_NAME}-jenkins:${BUILD_NUMBER}'
                        sh 'docker tag romeo23/server-${BRANCH_NAME}-jenkins:${BUILD_NUMBER} romeo23/server-${BRANCH_NAME}-jenkins:latest'
                        sh 'docker push romeo23/server-${BRANCH_NAME}-jenkins:latest'
                    }
                }
            }
        }
        stage('Kubernetes deploy') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f server-deployment.yaml'
                    sh 'kubectl apply -f server-service.yaml'
                }
            }
        }
    }
}
