pipeline {
    agent any 

    tools {
        maven "m3"
    }

    environment {
        SONAR_SERVER_URL = "http://52.87.232.142:9000"
        SONAR_PROJECT_KEY = "hello-world-cicd"
        SONAR_PROJECT_NAME = "hello-world-cicd"
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
    }
}
