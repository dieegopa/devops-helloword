pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'No hay que compilar nada, esto es python, instalo pytest y flask porque estoy en mac'
                sh 'ls -la'
                sh 'echo $WORKSPACE'
                sh '''
                python3 -m venv venv
                source venv/bin/activate
                export PYTHONPATH=$WORKSPACE
                pip install pytest
                pip install flask
                '''
            }
        }
        stage('Test'){
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                            source venv/bin/activate
                            export PYTHONPATH=$WORKSPACE
                            pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
                stage('Services') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh '''
                            source venv/bin/activate
                            export FLASK_APP=app/api.py
                            nohup flask run > flask.log 2>&1 &
                            nohup java -jar test/wiremock.jar --port 9090 --root-dir test/wiremock > wiremock.log 2>&1 &
                            sleep 5
                            export PYTHONPATH=$WORKSPACE
                            pytest --junitxml=result-rest.xml test/rest
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