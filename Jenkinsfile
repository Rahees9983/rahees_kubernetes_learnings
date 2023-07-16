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
                        docker build -t 34.133.208.85:8083/springapp:${VERSION} .
                        docker login -u $docker_username -p $docker_password 34.133.208.85:8083
                        docker push 34.133.208.85:8083/springapp:${VERSION}
                        docker rmi 34.133.208.85:8083/springapp:${VERSION}
                        '''
        }
        }
        }
    }
    // stage('indentifying misconfigs using datree in helm charts'){
    //         steps{
    //             script{

    //                 dir('kubernetes/') {
    //                     // withEnv(['DATREE_TOKEN=GJdx2cP2TCDyUY3EhQKgTc']) {
    //                           sh 'helm datree test myapp/'
    //                     // }
    //                 }
    //             }
    //         }
    //     }

    stage("pushing the helm charts to nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                          dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tgz myapp/
                                 curl -u admin:$docker_password http://34.133.208.85:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                          }
                    }
                }
            }
        }
    stage('manual approval'){
        steps{
            script{
                    timeout(10){
                    mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "khanrahees333@gmail.com";  
                    input(
                    message: "Continue to Deploy?",
                    ok: "Deploy"
                )
                    }
                }
        }
    }

    stage('Deploying application on k8s cluster') {
            steps {
               script{
                   kubeconfig(credentialsId: 'pankaj-k8s-kube-config', serverUrl: 'https://10.128.0.31:6443') {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.133.208.85:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
                    }
               }
            }
        }

    }
    //  post {
	// 	always {
	// 		mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "khanrahees333@gmail.com";  
	// 	 }
	//    }
}