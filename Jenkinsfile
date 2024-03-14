pipeline {
    agent any 

    tools {
        maven "m3"
    }

    stages {
        stage("code-checkout") {
            git 'https://github.com/ash2code/hello-world-cicd.git'
        }

        stage("code-build") {
            steps {
                script {
                    sh "mvn install package"
                }
            }
        }
    }
}
