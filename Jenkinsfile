pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            agent {
                any {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
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
            withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password'), string(credentialsId: 'docker_username', variable: 'docker_username')]) {
                     sh '''
                        sudo docker build -t 34.133.208.85:8083/springapp:${VERSION} .
                        sudo docker login -u $docker_username -p $docker_password 34.133.208.85:8083
                        sudo docker push 34.133.208.85:8083/springapp:${VERSION}
                        sudo docker rmi 34.133.208.85:8083/springapp:${VERSION}
                        '''
        }
        }
        }
    }
    }
}