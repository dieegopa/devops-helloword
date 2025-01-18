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
                            python3 -m coverage run --branch --source=app --omit=app/init_py,app/api.py -m pytest --junitxml=result-unit.xml test/unit
                            '''
                            junit 'result-unit.xml'
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
                            junit 'result-rest.xml'
                        }
                    }
                }
                stage('Static') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh 'python3 -m flake8 --format=pylint --exit-zero app > flake8.out'
                            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[integerThreshold: 8, threshold: 8.0, type: 'TOTAL'], [criticality: 'ERROR', integerThreshold: 10, threshold: 10.0, type: 'TOTAL']]
                        }
                    }
                }
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