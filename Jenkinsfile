pipeline {
    agent any
    options { skipDefaultCheckout() }
    stages {
        stage('Get Code') {
            steps {
                echo '---- CLEAN BEFORE BUILD STARTS ----'
                cleanWs()
                echo '---- DOWNLOAD REPO ----'
                checkout scm
                echo '---- WORKSPACE ----'
                echo WORKSPACE
            }
        }

        stage('Flake8') {
            steps {
                bat '''
                    flake8 --exit-zero --format=pylint app >flake8.out
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            }
        }

        stage('Security') {
            steps {
                bat '''
                    bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
            }
        }

        stage('Unit') {
            steps {
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                    coverage run --branch --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                '''
                junit 'result-unit.xml'
            }
        }

        stage('Rest') {
            steps {
                bat '''
                    set FLASK_APP=app\\api.py
                    start flask run -p 5001
                    start java -jar C:\\Unir\\Ejercicios\\wiremock-jre8-standalone-2.28.0.jar --port 9090 --root-dir test\\wiremock
                    set PYTHONPATH=.
                    ping 127.0.0.1 -n 15
                    pytest --junitxml=result-rest.xml test\\rest
                '''
                junit 'result-rest.xml'
            }
        }

        stage('Cobertura') {
            steps {
                bat '''
                    coverage xml
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    recordCoverage qualityGates: [[criticality: 'ERROR', integerThreshold: 85, metric: 'LINE', threshold: 85.0], [integerThreshold: 95, metric: 'LINE', threshold: 95.0], [criticality: 'ERROR', integerThreshold: 80, metric: 'BRANCH', threshold: 80.0], [integerThreshold: 90, metric: 'BRANCH', threshold: 90.0]], tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']]
                }
            }
        }

        stage('Performance') {
            steps {
                bat '''
                    set FLASK_APP=app/api.py
                    set FLASK_ENV=development
                    start flask run

                    ping 127.0.0.1 -n 15

                    C:\\UNIR\\Ejercicios\\apache-jmeter-5.5\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}
