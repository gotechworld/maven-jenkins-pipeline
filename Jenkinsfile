pipeline {
  agent any
  tools {
        maven "Maven 3.8.6" 
   }

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }  
       }
      stage('Test Maven - JUnit') {
            steps {
              sh "mvn test"
            }
            post{
              always{
                junit 'target/surefire-reports/*.xml'
              }
            }
        }


      stage('SonarCloud Analysis - SAST') {
                steps {
                      withSonarQubeEnv('SonarCloud') {
                         sh "mvn sonar:sonar \
                                      -Dsonar.projectKey=maven-jenkins-pipeline \
                                      -Dsonar.organization=maven-jenkins-pipeline \
                                      -Dsonar.host.url=https://sonarcloud.io \
                                      -Dsonar.login=80b21f2c7896dafe2426aa475bf32f7fbb2ddce7" 
                    }
                 }
      }              



      stage("Quality Gate") {
            steps {
              timeout(time: 15, unit: 'MINUTES') { // If analysis takes longer than indicated time, then build will be aborted
                  waitForQualityGate abortPipeline: true
                  script{
                      def qg = waitForQualityGate() // Waiting for analysis to be completed
                      if(qg.status != 'OK'){ // If quality gate was not met, then present error
                          error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                  }
              }
            }
      }
  }
}
