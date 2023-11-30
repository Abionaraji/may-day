pipeline{
    agent any
    tools{
        maven 'Maven'
        jdk 'JDK'
    }
    stages{
        stage('Git Checkout'){
            steps{
                git branch: 'ci-jenkins', url: 'https://github.com/Abionaraji/personally-project.git'
            }
        }
        stage('Maven Build'){
            steps{
                sh 'mvn clean install'
            }
            post {
                success{
                    echo 'New achiving'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Unit Test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Checkstyle Analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage('Integrated Testing'){
            steps{
                sh 'mvn verify -DiskipUnitTests'
            }
        }
        stage('Sonar Scanner'){
            steps{
                withSonarQubeEnv(credentialsId: 'jenkins-sonar', installationName: 'SonarQube') {
                    sh 'cat .scannerwork/report-task.txt'
// copy to a properties file so we can ingest as variables
sh 'cp .scannerwork/report-task.txt .scannerwork/report-task.properties'
def props = readProperties file: '.scannerwork/report-task.properties'
def ceTaskUrl= props['ceTaskUrl']
def sonarServerUrl=props['serverUrl']
// wait for analysis to complete
waitUntil {
    def response = httpRequest consoleLogResponseBody: true, contentType: 'APPLICATION_JSON', ignoreSslErrors: true, url: "${ceTaskUrl}", wrapAsMultipart: false
    println "Sent a request, got a $response response"
    def body = readJSON text: response.content
    body.each { key, value -> }
    ceTask = body.task.status
    echo "Status is "+ceTask
    if ("FAILED".equals(ceTask)){
    echo "failed = "+ceTask
    throw new Exception(failed+" Sonar process failed!")
    }
    return "SUCCESS".equals(ceTask)
    sleep 30
         }
                }
            }
        }
    }
}
