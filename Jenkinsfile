pipeline {
    agent any 

    tools {
        maven "m3"
    }

    environment {
        SONAR_SERVER_URL = "http://100.25.17.74:9000"
        SONAR_PROJECT_KEY = "hello-world-cicd"
        SONAR_PROJECT_NAME = "hello-world-cicd"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "100.25.17.74:8081"
        NEXUS_REPOSITORY = "hello-world-cicd"
        NEXUS_CREDENTIAL_ID = ""
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage("code-checkout") {
            steps {
            git 'https://github.com/ash2code/hello-world-cicd.git'
        }
        }

        stage("code-build") {
            steps {
                script {
                    sh "mvn install package"
                }
            }
        }

        stage("SonarQube-Analysis") {
            steps {
                script {
                    sh """ 
                    mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${env.SONAR_PROJECT_NAME} \
                        -Dsonar.host.url=${env.SONAR_SERVER_URL} \
                        -Dsonar.token=sqp_8e399fa8886ee929f778f16d555f35f18ad31a27
                    """
                }
            }
        }

        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
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
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
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

      
       
