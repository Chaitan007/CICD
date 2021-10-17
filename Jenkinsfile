pipeline{
    agent any

    stages{
        stage("SonarQube Quality check"){
            agent {
                docker {
                    image "openjdk:11"
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'SonarQube-Pass') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }

                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                      }
                    }

                }

            }
        }
    }
    post{
        always{
            echo " Build is success"

        }
    }
}