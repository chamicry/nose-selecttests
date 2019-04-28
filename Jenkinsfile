pipeline {
    environment {
        docker_image_name = "python3-unittests"
        HTTP_PROXY = "${params.HTTP_PROXY}"
        JENKINS_USER_ID = "${params.JENKINS_USER_ID}"
        JENKINS_GROUP_ID = "${params.JENKINS_GROUP_ID}"
    }
    agent {
        dockerfile {
            additionalBuildArgs '--build-arg "JENKINS_USER_ID=112" --build-arg "JENKINS_GROUP_ID=117"'
            filename 'Dockerfile.build'
            dir '.'
            label env.docker_image_name
        }
    }

    stages {

        stage('静的解析') {
            steps {
                parallel(
                    'Pep8': {
                        dir('.') {
                            script {
                                sh 'pep8 . --exclude=**/test*.py |true'
                                sh 'set HTTP_PROXY=$HTTP_PROXY'
                                sh 'set HTTPS_PROXY=$HTTP_PROXY'
                            }
                            step([
                                $class: 'WarningsPublisher',
                                consoleParsers: [
                                    [parserName: 'Pep8']
                                ]
                            ])
                        }
                    }
                )
            }
        }

        stage('ユニットテスト') {
            steps {
                script {
                    dir('.') {
                        sh 'nosetests -v --with-xunit --with-coverage --cover-inclusive --cover-erase --cover-tests --cover-branches  --cover-xml --cover-package=functional_tests,noseselecttests || true'
                    }
                }
                step([
                    $class: 'JUnitResultArchiver',
                    testResults: 'nosetests.xml'
                ])
                step([
                    $class: 'CoberturaPublisher',
                    autoUpdateHealth: false,
                    autoUpdateStability: false,
                    coberturaReportFile: 'coverage.xml',
                    failUnhealthy: false,
                    failUnstable: false,
                    maxNumberOfBuilds: 0,
                    onlyStable: false,
                    zoomCoverageChart: false
                ])
                archiveArtifacts 'nosetests.xml'
                archiveArtifacts 'coverage.xml'
            }
        }

    }
}
