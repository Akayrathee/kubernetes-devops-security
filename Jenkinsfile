pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so 
            }
        }   
      stage('Unit Tests - JUnit and Jacoco') {
            steps {
              sh "mvn test"
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
    
        } 
      stage('Docker Build and Push'){
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]){
          sh 'printenv'
          sh 'docker build -t aakashrathee/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push aakashrathee/numeric-app:""$GIT_COMMIT""'   //git added a new stage
      }
      }
    }
      stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#aakashrathee/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }

    }
  }
    
}