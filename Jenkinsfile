pipeline {
  agent { label "linux" }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  tools {
        maven "Maven 3.8.6" 
   }

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
            post {
              success {
                echo 'Successfully build.'
              }

              failure {
                echo 'Failed to build.'
              }
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


      stage('SAST') {
                steps {
                      withSonarQubeEnv('SonarQubeDocker') {
                         sh "mvn clean verify sonar:sonar \
                              -Dsonar.projectKey=maven-jenkins-pipeline \
                              -Dsonar.host.url=http://157.245.71.113:9000 \
                              -Dsonar.login=sqp_f0859583c91629ba63a6e0bc8d9b071b5f644437"
                              
                      }
                }
                post {
                    success {
                      echo 'Successfully scanned.'
                    }

                    failure {
                      echo 'Failed to scan.'
                    }
            }
      }              



      stage('Quality Gate') {
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
            post {
              success {
                echo 'Successfully QG passed.'
              }

              failure {
                echo 'Failed to pass.'
              }
            }
      }
  }
}
