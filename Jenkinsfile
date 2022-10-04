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
           sh "mvn -B verify sonar:sonar \
                        -Dsonar.projectKey=maven-jenkins-pipeline \
                        -Dsonar.organization=maven-jenkins-pipeline \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=80b21f2c7896dafe2426aa475bf32f7fbb2ddce7" 
                }
           timeout(time: 2, unit: 'MINUTES') {
                      script {
                        waitForQualityGate abortPipeline: true
                    }
                }
              }
        }
     }
}
