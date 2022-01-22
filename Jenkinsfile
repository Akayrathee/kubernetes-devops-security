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
      stage('Mutation Tests - PIT') {
          steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
          }
      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      }
    //   stage('SonarQube - SAST') {
    //   steps {
    //     withSonarQubeEnv('sonarcube'){
    //     sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-aakash.eastus.cloudapp.azure.com:9000 -Dsonar.login=ef24409f636e6a68a31fdc65f8d1ea103b0d75bf"
    //     }
    //     timeout(time: 2, unit: 'MINUTES'){
    //       script{
    //         waitForQualityGate abortPipeline: true
    //       }
    //     }
    //   }
    // }

    stage('SonarQube - SAST') {   //WebHook Sahi kia
      steps {
        withSonarQubeEnv('SonarCube') {
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://devsecops-aakash.eastus.cloudapp.azure.com:9000 -Dsonar.login=ef24409f636e6a68a31fdc65f8d1ea103b0d75bf"
        }
        timeout(time: 8, unit: 'MINUTES') {
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }
    // Added Vulnerable version checking
    stage('Vulnerability Scan - Docker ') {
      steps {
        sh "mvn dependency-check:check"
      }
      post {
        always {
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
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