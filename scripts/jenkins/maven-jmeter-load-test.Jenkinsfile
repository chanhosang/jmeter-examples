/*
 ============================================================================
 This Jenkins declarative pipeline (using docker pipeline plugin) demonstrate how to run
 load test (Integration of Selenium with JMeter) with Taurus Tool inside a container build agent.
 The image is based on the official blzemeter/taurus image from Docker Hub.
 ============================================================================

 Pre-requisite:
- Installed the following jenkins plugins:
	* https://wiki.jenkins.io/display/JENKINS/Git+Plugin
    * https://wiki.jenkins.io/display/JENKINS/Docker+Plugin
	* http://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin
*/

def jenkins = [:]

String sectionHeaderStyle = '''
    background: LightGreen;
    text-align: center;
'''

String separatorStyle = '''
    border: none;
'''

properties(
  [
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')),
    parameters([
        [
            $class: 'ParameterSeparatorDefinition',
            sectionHeader: 'Thread Group Setting',
            separatorStyle: separatorStyle,
            sectionHeaderStyle: sectionHeaderStyle
        ],
        string(name: "number_of_threads", defaultValue: "2", description: "Specify the even number of concurrent threads (e.g. 2, 4, 10, 50 or 100) to execute the test from the two thread groups configured in the project."),
        string(name: "ramp_up", defaultValue: "2", description: 'Specify how long (seconds) to take to ramp-up to the full number of threads specified in "number_of_threads" parameter.'),
        string(name: "loop_count", defaultValue: "2", description: 'Specify the number of iterations for each thread.')

      ]
    )
  ]
)

pipeline {
    agent {
        label 'master'
    }

    options {
        // disableConcurrentBuilds()
        timestamps()
        skipDefaultCheckout()
    }

    stages {

        stage('Preparation') {
            steps {
                // Enabled skipDefaultCheckout()
                script {
                    // sh 'printenv'

                    // checkout from version control in here
                    // git branch: 'main', credentialsId: '<credentialsId>', url: '<repositoryUrl>'

                    // checkout from version control configured in pipeline job
                    echo "git url: ${scm.userRemoteConfigs[0].url}"
                    echo "git branch: ${scm.branches[0].name}"
                    checkout scm

                    // Change build name
                    currentBuild.displayName = "${currentBuild.number}:T${params.number_of_threads}-R${params.ramp_up}-L${params.loop_count}"
                }

                stash name: 'source', includes: 'jmeter/**'
            }
		}

        stage('Run Load Testing') {
            agent {
                docker {
                    image 'maven:3-jdk-8'
                    label 'master'
                    args '-v $HOME/.m2:/root/.m2 --entrypoint='
                    // args '--entrypoint=' //args '-u root --entrypoint=\'/bin/sh\''
                }
            }
            steps {

                unstash "source"

                echo 'Run load test with Maven, generate and publish reports'

                script {

                    // -o modules.jmeter.properties=\"{'jmeter.reportgenerator.overall_granularity':1000}\"
                    sh """
                    mvn clean verify -P jmeter_load_test_purchase_product \
                    -f jmeter/pom.xml \
                    -DnumberOfThreads=${params.number_of_threads} -DrampUp=${params.ramp_up} -DloopCount=${params.loop_count}
                    """

                }
            }

			post {
                always {

                    sh """
                    target_dir="jmeter/target/jmeter/testFiles"; if [ -d "\${target_dir}" ]; then rm -r \${target_dir}; fi
                    target_dir="jmeter/target/jmeter/custom_properties"; if [ -d "\${target_dir}" ]; then rm -r \${target_dir}; fi
                    target_dir="jmeter/target/jmeter/lib"; if [ -d "\${target_dir}" ]; then rm -r \${target_dir}; fi
                    target_dir="jmeter/target/jmeter/bin"; if [ -d "\${target_dir}" ]; then rm -r \${target_dir}; fi
                    """

                    // Archive artifacts
                    sh """
                    tar -czvf B${currentBuild.number}_T${params.number_of_threads}_R${params.ramp_up}_L${params.loop_count}.tar.gz jmeter/target/jmeter
                    """

                    archiveArtifacts artifacts: "**/*.tar.gz", fingerprint: true

                    perfReport errorFailedThreshold: 50,
                    errorUnstableThreshold: 10,
                    filterRegex: '',
                    sourceDataFiles: 'jmeter/target/jmeter/results/*.csv'

                    publishHTML([allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false, reportDir: "jmeter/target/jmeter/reports",
                    reportFiles: "**/index.html",
                    reportName: 'HTML Report', reportTitles: ''])

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
