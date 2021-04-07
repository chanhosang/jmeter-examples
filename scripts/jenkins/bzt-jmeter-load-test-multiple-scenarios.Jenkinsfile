/*
 ============================================================================
 This Jenkins declarative pipeline (using docker pipeline plugin) demonstrate how to run
 load test with Taurus Tool inside a container build agent.
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
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '20')),
    parameters([
        [
            $class: 'ParameterSeparatorDefinition',
            sectionHeader: 'Primary Scenario Load Setting',
            separatorStyle: separatorStyle,
            sectionHeaderStyle: sectionHeaderStyle
        ],
        string(name: "primary_concurrency", defaultValue: "1", description: 'How many concurrent connections to execute the test?'),
        string(name: "primary_iterations", defaultValue: "1", description: 'Specify the number of times (iteration) to execute the test.'),
        string(name: "primary_ramp_up", defaultValue: "1s", description: 'If you are running more than 1 thread, please increase the ramp-up period accordingly.'),
        [
            $class: 'ParameterSeparatorDefinition',
            sectionHeader: 'Secondary Scenario Load Setting',
            separatorStyle: separatorStyle,
            sectionHeaderStyle: sectionHeaderStyle
        ],
        string(name: "secondary_concurrency", defaultValue: "1", description: 'How many concurrent connections to execute the test?'),
        string(name: "secondary_duration", defaultValue: "1m", description: 'Specify the duration of the test.'),
        string(name: "secondary_ramp_up", defaultValue: "1s", description: 'If you are running more than 1 thread, please increase the ramp-up period accordingly.'),

        [
            $class: 'ParameterSeparatorDefinition',
            sectionHeader: 'Flags',
            separatorStyle: separatorStyle,
            sectionHeaderStyle: sectionHeaderStyle
        ],
        booleanParam(
            description: "Enable to send interim and final test result to BlazeMeter Service (https://a.blazemeter.com/).",
            name: "run_blazemeter_reporter",
            defaultValue: false
        )

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
                    currentBuild.displayName = "${currentBuild.number}:T${params.concurrency}-R${params.ramp_up}-L${params.iterations}"
                }

                stash name: 'source', includes: 'scripts/bzt/**,jmeter/**'
            }
		}


        stage('Run Load Testing') {
            agent {
                docker {
                    image 'blazemeter/taurus:latest'
                    label 'master'
                    args '-v $HOME/.bzt:/root/.bzt --entrypoint=' //args '-u root --entrypoint=\'/bin/sh\''
                }
            }
            steps {

                unstash "source"

                echo 'Run load test with BlazeMeter Taurus, generate and publish reports'
                // docker run --rm \
                // -v /home/ubuntu/bzt/bzt-examples:/bzt-configs \
                // -v /home/ubuntu/bzt/bzt-results:/tmp/artifacts \
                // blazemeter/taurus bzt-jmeter-load-test.yml -o settings.env.RESULTS_DIR=results

                script {
                    def extra_args = ""

                    // If true, uploads tests stats into BlazeMeter.com service
                    if (params.run_blazemeter_reporter) {
                        extra_args = "-report"
                    }

                    // -o modules.jmeter.properties=\"{'jmeter.reportgenerator.overall_granularity':1000}\"
                    sh """
                    bzt scripts/bzt/bzt-jmeter-load-test-multiple-scenarios.yml \
                    scripts/bzt/bzt-jmeter-load-test-multiple-scenarios-reporting.yml \
                    -o execution.0.concurrency=${params.primary_concurrency} \
                    -o execution.0.ramp-up=${params.primary_ramp_up} \
                    -o execution.0.iterations=${params.primary_iterations} \
                    -o execution.1.concurrency=${params.secondary_concurrency} \
                    -o execution.1.ramp-up=${params.secondary_ramp_up} \
                    -o execution.1.hold-for=${params.secondary_duration} \
                    ${extra_args}
                    """
                    // Move the reports from taurus artifacts dir location to workspace dir
                    sh """
                    mv /tmp/artifacts .
                    tar -zcvf logs.tar.gz **/*.log
                    tar -zcvf taurus-artifacts-${currentBuild.number}.tar.gz artifacts
                    """
                }
            }

			post {
                always {
                    archiveArtifacts artifacts: "**/*.tar.gz", fingerprint: true

                    perfReport errorFailedThreshold: 50,
                    errorUnstableThreshold: 10,
                    filterRegex: '',
                    sourceDataFiles: '**/results/results.xml'

                    publishHTML([allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'artifacts/reports',
                        reportFiles: '**/index.html',
                        reportName: 'HTML Report',
                        reportTitles: ''])
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
