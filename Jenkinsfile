pipeline{
    agent any
    tools{
        maven "MAVEN3.9"
        jdk "JAVA17"
    }
    environment {
        registryCredential = 'DockerHubSecret'
        registry = "prajwalgouthammp/vprofile-app-prod-image"
    }
    stages{
        stage('Build'){
            steps{
                sh 'mvn install -DskipTests'
            }
        }
        stage('Unit test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('check style analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('Sonarqube code Analysis'){
        environment {
        scannerHome = tool 'SONAR6.2'  // Reference the SonarQube Scanner tool
         }
        steps{
              withSonarQubeEnv('SonarServer') {
                sh '''
                   ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                   '''

            }
        }
    }
    // stage('Quality Gate Checking') {
    //         steps {
    //             script {
    //                 timeout(time: 1, unit: 'HOURS') {
    //                     waitForQualityGate abortPipeline: true
    //                 }
    //             }
    //         }
    //     }
        stage('Build App Image') {
          steps {
            script {
                dockerImage = docker.build( registry + ":$BUILD_NUMBER", "./")
                }
          }
    
        }

        stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( '', registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
          post{
            success {
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
          }
        }
        stage('Kubernetes Deploy') {
          agent { label 'KOPS-SLAVE' }
            steps {
                    sh "helm install vproifle-stack helm-vprofile/vprofile-helm-charts --set appimage=${registry}:${BUILD_NUMBER} --namespace prod"
            }
        }
      

}
post {
        always {
            script {
                def buildStatus = currentBuild.result ?: 'SUCCESS'
                slackSend channel: '#vprofile',
                          color: buildStatus == 'SUCCESS' ? 'good' : 'danger',
                          message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${buildStatus} \n More info at: ${env.BUILD_URL}"
            }
        }
    }
}
