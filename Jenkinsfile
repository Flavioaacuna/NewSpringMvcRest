pipeline {
        agent any
        stages {
            stage('Build') {
            steps {
            sh 'mvn -B package'

        }
}
stages {
    stage('build user') {
      steps {
        wrap([$class: 'BuildUser']) {
          sh 'echo "${BUILD_USER}"'
        }
      }
    }
  }
            stage('SonarQube analysis') {
            environment {
            SCANNER_HOME = tool 'sonarqube'
        }
                steps {
                withSonarQubeEnv(credentialsId: 'Sonar', installationName: 'Sonarqube') {
                sh '''$SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectKey=projectKey \
                -Dsonar.projectName=ProyectoGrupo3\
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
                    nexusUrl: "192.168.23.60:8081",
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


    post{
        always{
               
            slackSend( channel: "#notificaciones-desde-jenkins", token: "slack trabajo grupal", color: "good", username: "Admin", message: "${custom_msg()}")
            }
    }
}


def custom_msg()
{
def JENKINS_URL= "192.168.23.60:8080"
def JOB_NAME = env.JOB_NAME
def BUILD_ID= env.BUILD_ID
def JENKINS_LOG= "Se ha realizado un Build: ${env.JOB_NAME} ${env.BUILD_DISPLAY_NAME} (<${env.BUILD_URL}|Revisar-estado>)"
return JENKINS_LOG
}
