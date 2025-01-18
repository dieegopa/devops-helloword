pipeline {
    agent any
    stages {
        stage('Test'){
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                            export PYTHONPATH=$WORKSPACE
                            python3 -m pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
                stage('Services') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                            export PYTHONPATH=$WORKSPACE
                            export FLASK_APP=app/api.py
                            nohup python3 -m flask run > flask.log 2>&1 &
                            nohup java -jar test/wiremock.jar --port 9090 --root-dir test/wiremock > wiremock.log 2>&1 &
                            sleep 10
                            python3 -m pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        stage('Results') {
            steps {
                junit 'result*.xml'
            }
        }
    }

    post {
        always {
            sh '''
            # Finalizar procesos de Flask y WireMock
            pkill -f "flask run" || true
            pkill -f "wiremock" || true
            '''
        }
    }
}