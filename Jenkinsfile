pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
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
                            '''

                    }       
                }
            }

        }
            stage("Pushing helm charts to nexus"){
            steps {
                script{
                    withCredentials([string(credentialsId: 'docker_login_pass', variable: 'docker_login_pass')]) {
                          dir('kubernetes/') {
                            sh '''
                                helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                tar -czvf myapp-${helmversion}.tgz myapp/
                                curl -u admin:$docker_login_pass http://157.245.37.88:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v

                            '''
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