pipeline {
      environment {
        SLACK_CHANNEL = "#modulo3leccion7"
        SLACK_TEAM_DOMAIN = "Sustantiva"
        SLACK_TOKEN = credentials("Token-Slack")
    }
    agent any
        stages {
         
        stage('Build') {
            steps {
                 sh "mvn -B package"
            }
        } 
         stage('SonarQube analysis') {
            environment {
            SCANNER_HOME = tool 'sonarqube1'
        }
                steps {
                withSonarQubeEnv(credentialsId: 'Sonar', installationName: 'Sonarqube') {
                sh '''$SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectKey=projectKey \
                -Dsonar.projectName=Modulo3L6\
                -Dsonar.sources=src/ \
                -Dsonar.java.binaries=target/classes/ \
                -Dsonar.exclusions=src/test/java/****/*.java \
                -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}'''
        }
    }
}
            
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: "nexus3",
                            protocol: "http",
                            nexusUrl: "172.20.212.68:8081",
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: "Host",
                            credentialsId: "pass1",
                            artifacts: [
                                [artifactId: pom.artifactId,
                                        classifier: '',
                                        file: artifactPath,
                                        type: pom.packaging]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        
            } 
     }
    post {
        always {
                slackSend message:"Regardless of the completion status of the Pipeline or stage run - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        changed {
                slackSend message:"the current Pipeline run has a different completion status from its previous run - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        fixed {
                slackSend message:"The current Pipeline run is successful and the previous run failed or was unstable - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        regression {
                slackSend message:"The current Pipeline or status is failure, unstable, or aborted and the previous run was successful - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        aborted {
                slackSend message:"The current Pipeline run has an <aborted> status was successful - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        failure {
                slackSend message:"The current Pipeline or stage run has a <failed> status  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
        success {
                slackSend message:"The current Pipeline or stage run has a <success> status - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        unstable {
                slackSend message:"The current Pipeline run has an <unstable> status - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        unsuccessful {
                slackSend message:"The current Pipeline or stage run has not a <success> status - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
        cleanup {
                slackSend message:"Every other post condition has been evaluated - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }            
    }
}


def custom_msg()
{
def JENKINS_URL= "172.20.212.68:8080"
def JOB_NAME = env.JOB_NAME
def BUILD_ID= env.BUILD_ID
def JENKINS_LOG= " FAILED: Job [${env.JOB_NAME}] Logs path: ${JENKINS_URL}/job/${JOB_NAME}/${BUILD_ID}/The current Pipeline or stage run has a <failed> status"
return JENKINS_LOG
}
