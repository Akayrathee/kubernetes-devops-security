pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so 
            }
        }   
      stage('Unit Tests') {   // Added this stage block here in video 26
            steps {
              sh "mvn test"
              
            }
        } 
    }
}