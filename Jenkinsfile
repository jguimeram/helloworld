pipeline {
    agent any

    stages {
        stage('Download'){
            steps{
                echo "---- DOWNLOAD REPO ----"
                checkout scm
                //git url: "https://github.com/jguimeram/helloworld"
                sh "ls -la"
                echo "---- WORKSPACE ----"
                echo WORKSPACE
            }
        }
        stage ('Test'){
            parallel{
                stage('Unit'){
                     steps{
                        echo "---- UNIT ----"
                        sh '''
                        export PYTHONPATH=$PWD
                        pytest ./test/unit
                        '''
                     }
                }
                stage('Service'){
                    steps{
                        echo "---- SERVICE ----"
                        sh'''
                        export FLASK_APP=./app/api.py
                        flask run &
                        java -jar ./test/wiremock/wiremock-standalone-3.13.2.jar --port 9090 --root-dir ./test/wiremock &
                        sleep 10s
                        pytest --junitxml=result-rest.xml test/rest
                        '''
                    }
                }
            }
        }
    }
}
