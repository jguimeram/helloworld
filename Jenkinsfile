pipeline {
    agent any
    options { skipDefaultCheckout() }
    stages {
        stage('Get Code') {
            steps {
                echo '---- DOWNLOAD REPO ----'
                checkout scm
                echo '---- WORKSPACE ----'
                echo WORKSPACE
            }
        }

        stage('Test') {
            parallel {
                stage('Unit') {
                    steps {
                        echo '---- UNIT ----'
                        sh '''
                        export PYTHONPATH=$PWD
                        pytest ./test/unit
                        '''
                    }
                }
                stage('Rest') {
                    steps {
                        echo '---- REST ----'
                        sh'''
                        export FLASK_APP=./app/api.py
                        flask run &
                        java -jar /home/jenkins/wiremock/wiremock-standalone-3.13.2.jar --port 9090 --root-dir ./test/wiremock &
                        sleep 3s
                        pytest --junitxml=result-rest.xml test/rest
                        '''
                    }
                }

                stage('Static') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                        echo '---- STATIC ----'
                        sh'''
                        export PYTHONPATH=$PWD
                        flake8 --format=pylint --exit-zero app > flake8.out
                        '''
                        recordIssues qualityGates: [[integerThreshold: 8, threshold: 8.0, type: 'TOTAL'], [criticality: 'FAILURE', integerThreshold: 10, threshold: 10.0, type: 'TOTAL']], sourceCodeRetention: 'LAST_BUILD', tools: [flake8(pattern: 'flake8.out')]
                        }
                    }
                }

                stage('Coverage') {
                    steps {
                        sh'''
                        coverage run --branch --source=app --omit="app/__init__.py,app/api.py" -m pytest test/unit
                        coverage xml
                        '''
                        recordCoverage qualityGates: [[criticality: 'ERROR', integerThreshold: 85, metric: 'LINE', threshold: 85.0], [criticality: 'NOTE', integerThreshold: 95, metric: 'LINE', threshold: 95.0], [criticality: 'ERROR', integerThreshold: 80, metric: 'BRANCH', threshold: 80.0], [criticality: 'NOTE', integerThreshold: 90, metric: 'BRANCH', threshold: 90.0]], tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']]
                    }
                }

                stage('Security') {
                    steps {
                        sh'''
                        bandit --exit-zero -r test -f custom -o bandit.out
                        '''
                        recordIssues qualityGates: [[integerThreshold: 1, threshold: 1.0, type: 'TOTAL'], [criticality: 'FAILURE', integerThreshold: 3, threshold: 3.0, type: 'TOTAL']], sourceCodeRetention: 'LAST_BUILD', tools: [pyLint(name: 'bandit', pattern: 'bandit.out')]
                    }
                }

                stage('Performance') {
                    steps {
                        sh'''
                        rm -vf /home/jenkins/scripts/test1.jtl
                        /home/jenkins/jmeter/bin/jmeter -n -t /home/jenkins/scripts/test1.jmx -l /home/jenkins/scripts/test1.jtl
                        '''
                        perfReport filterRegex: '', showTrendGraphs: true, sourceDataFiles: '/home/jenkins/scripts/test1.jtl'
                    }
                }
            }
        }
    }
    post {
        always {
            echo '---- Clean Workspace ----'
            deleteDir()
        }
    }
}


