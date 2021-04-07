// Please install the following plugins, and restart jenkins:
// - Docker Pipeline
// - Pipeline
// - Git
// - Workspace Cleanup

pipeline {

    agent {
        label 'master'
    }

    stages {

        stage('Run Maven') {
            agent {
                docker {
                    image 'maven:3-jdk-8'
                    label 'master'
                    args '-v $HOME/.m2:/root/.m2 --entrypoint='
                    // args '--entrypoint=' //args '-u root --entrypoint=\'/bin/sh\''
                }
            }
            steps {

                echo 'Run Maven Command'

                script {
                    sh """
                    mvn --version
                    """
                }
            }

			post {
                always {

                    cleanWs()
                }
            }
        }

        stage('Run Taurus') {
            agent {
                docker {
                    image 'blazemeter/taurus:latest'
                    label 'master'
                    args '-v $HOME/.bzt:/root/.bzt --entrypoint=' //args '-u root --entrypoint=\'/bin/sh\''
                }
            }
            steps {

                echo 'Run BlazeMeter Taurus'

                script {
                    sh """
                    bzt -h
                    """
                }
            }

			post {
                always {
                    cleanWs()
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
