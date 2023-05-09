pipeline {
    agent any
        stages {
         
        stage('Test') {
            steps {
                 sh "mvn clean verify sonar:sonar \
                 -Dsonar.projectKey=Sonar-jenkins \
                 -Dsonar.projectName='Sonar-jenkins' \
                 -Dsonar.host.url=http://172.22.134.190:9000 \
                 -Dsonar.token=sqp_a732c6fc7dd59e223310557619ae5fbb121f8bd6" 
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
                            nexusUrl: " 172.22.134.190:8081",
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
}
