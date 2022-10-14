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
                    withCredentials([string(credentialsId: 'nexus-password', variable: 'nexus_password')]) {
                             sh '''
                                docker build -t 34.131.139.168:8083/springapp:${VERSION} .
                                docker login -u admin -p ${nexus_password} 34.131.139.168:8083 
                                docker push  34.131.139.168:8083/springapp:${VERSION}
                                docker rmi 34.131.139.168:8083/springapp:${VERSION}
                            '''
                    }
                }
            }
        }
        stage('indentifying misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=8a021731-42c5-415d-abb4-93ec964e3b92']) {
                              sh 'helm datree test myapp/'
                        }
                          
                       }
                }
            }
                
        }
        stage("pushing helm chart to nexus"){
            steps{
                script{
                  dir('kubernetes/') {
                    withCredentials([string(credentialsId: 'nexus-password', variable: 'nexus_password')]) {
                             sh '''
                                helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                tar -czvf  myapp-${helmversion}.tgz myapp/
                                curl -u admin:{nexus_password} http://34.131.139.168:8081/repository/my-helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                    }
                    }
                }
            }
        }
        
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "shreyasshetty101@gmail.com";  
		 }
	   }
    
}