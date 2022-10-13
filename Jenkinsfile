pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
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
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-password', variable: 'nexus-password')]) {
                             sh '''
                                docker build -t 34.131.139.168:8083/springapp:${VERSION} .
                                docker login -u admin -p $nexus-password 34.131.139.168:8083 
                                docker push  34.131.139.168:8083/springapp:${VERSION}
                                docker rmi 34.131.139.168:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
        }
    }

    
