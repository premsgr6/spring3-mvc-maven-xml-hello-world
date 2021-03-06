pipeline {
    agent any
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
           maven "MAVEN_HOME" 
    }
    
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "ec2-3-136-22-211.us-east-2.compute.amazonaws.com:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "spring"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "NEXUS"
    }
    
    stages {
        stage('SCM') {
            steps {
                script {
                    git credentialsId: 'GIT-PASSWD', url: 'https://github.com/premsgr6/spring3-mvc-maven-xml-hello-world.git'
                }
            }
        }
        
        stage('MVN') {
            steps {
                script {
                    withMaven(jdk: 'JAVA', maven: 'MAVEN_HOME') {
                      sh label: '', script: 'mvn package'
                     }
                }
            }
        }
        
        stage('NEXUS') {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' and install pipeline-utility-steps plugin
                    pom = readMavenPom file: "pom.xml";
                    
                    // Find built artifact under target folder
                      filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // can be used bt this path aswell filesByGlob = findFiles(glob: "target/*.war");
                    
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: '${BUILD_NUMBER}',
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                     } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        
        
        stage('DEPLOY') {
            steps {
                script {
                    input message: 'wait for devops manager approval', submitter: 'sagar'
                     withCredentials([usernameColonPassword(credentialsId: 'QA-SERVER', variable: 'QA-SERVER')]) {
                     }
                    sh label: '', script: 'curl -v -u ${QA-SERVER} -T /var/lib/jenkins/workspace/pipeline2/target/spring3-mvc-maven-xml-hello-world-3.0-SNAPSHOT.war http://3.133.132.216:8080/manager/text/deploy?PATH=/spring2&update=true' 
                    
                }
            }
        }
    }
    
}
