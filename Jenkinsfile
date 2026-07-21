pipeline {
  agent any
  environment {
    ECR = "<637423492989>.dkr.ecr.eu-west-3.amazonaws.com"
    IMG = "${ECR}/vprofile-app:${BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Build + Test') {
      when { changeset "src/**" }
      steps { sh 'mvn -B clean package' }
    }
    stage('SonarQube') {
      when { changeset "src/**" }
      steps { sh 'mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN' }
    }
    stage('Docker Build') {
      when { anyOf { changeset "src/**"; changeset "docker/**" } }
      steps { sh 'docker build -t $IMG -f docker/app/Dockerfile .' }
    }
    stage('Trivy Scan') {
      when { anyOf { changeset "src/**"; changeset "docker/**" } }
      steps { sh 'trivy image --severity HIGH,CRITICAL --exit-code 1 $IMG' }
    }
    stage('Push to ECR') {
      when { anyOf { changeset "src/**"; changeset "docker/**" } }
      steps {
        sh 'aws ecr get-login-password | docker login --username AWS --password-stdin $ECR'
        sh 'docker push $IMG'
      }
    }
    stage('Deploy (Helm)') {
      when { anyOf { changeset "src/**"; changeset "helm/**" } }
      steps {
        sh 'aws eks update-kubeconfig --name vprofile-eks --region eu-west-3'
        sh '''helm upgrade --install vprofile ./helm/vprofile \
             -n vprofile --create-namespace \
             --set app.image.tag=${BUILD_NUMBER}'''
      }
    }
  }
}
