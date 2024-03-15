pipeline {
    agent any 

    tools {
        maven "m3"
    }

    environment {
        SONAR_SERVER_URL = "http://52.23.178.72:9000"
        SONAR_PROJECT_KEY = "hello-java"
        SONAR_PROJECT_NAME = "hello-java"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "http://52.23.178.72:8081" // Protocol should be included in the URL
        NEXUS_REPOSITORY = "hello-world-cicd"
        NEXUS_CREDENTIAL_ID = "nexus-user"
    }

    stages {
        stage("code-checkout") {
            steps {
                git 'https://github.com/ash2code/hello-world-cicd.git'
            }
        }

        stage("code-build") {
            steps {
                sh "mvn clean install" // Use 'clean install' instead of 'clean package' to avoid redundant build steps
            }
        }

        stage("SonarQube-Analysis") {
            steps {
                sh """ 
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                    -Dsonar.projectName=${env.SONAR_PROJECT_NAME} \
                    -Dsonar.host.url=${env.SONAR_SERVER_URL} \
                    -Dsonar.login=sqp_24efcbfebd34df82de6309a30ae0c515c5f727b5
                """
            }
        }

        stage("publish to nexus") {
            steps {
                echo "====++++  Publish to Nexus Repository Manager ++++===="
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

                    if (filesByGlob) {
                        def artifactPath = filesByGlob[0].path
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"

                        nexusArtifactUploader(
                            nexusVersion: env.NEXUS_VERSION,
                            protocol: env.NEXUS_PROTOCOL,
                            nexusUrl: env.NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: env.NEXUS_REPOSITORY,
                            credentialsId: env.NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        )
                    } else {
                        error "*** No files found matching pattern 'target/*.${pom.packaging}'"
                    }
                }
            }
        }
    }
}
