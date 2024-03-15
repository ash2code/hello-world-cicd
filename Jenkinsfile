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
