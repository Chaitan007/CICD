pipeline{
    agent any
    environment{
        VERSION = "$(env.BUILD_ID)"
    }

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
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }

                }

            }
        }
        stage("Docker build and push to Nexus"){
            steps {
                script{
                    withCredentials([string(credentialsId: 'docker_login_pass', variable: 'docker_login_pass')]) {
                            sh '''
                                docker build -t 157.245.37.88:8083/springapplication:${VERSION} .
                                docker login -u admin -p $docker_login_pass 157.245.37.88:8083
                                docker push 157.245.37.88:8083/springapplication:${VERSION}
                                docker rmi 157.245.37.88:8083/springapplication:${VERSION}

                    }       '''
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