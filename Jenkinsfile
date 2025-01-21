pipeline {
    agent any
    environment {
        pathscript = "C:\\Users\\ADM-1\\AppData\\Local\\Programs\\Python\\Python313\\Scripts"
        pathwiremock = "C:\\Users\\ADM-1\\Downloads\\wiremock-standalone-3.10.0.jar"
    }

    stages {
        stage('GetCode') {
            steps {
                git 'https://github.com/jlcalleja/helloworld'
            }
        }
        
        stage('Build') {
            steps {
                echo 'NO HAY QUE COMPILAR NADA, ESTO ES PYTHON'
                bat "dir"
                echo 'EL ESPACIO DE TRABAJO ES:'
                bat "echo %WORKSPACE%"
            }
        }
        
        stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                %pathscript%\\pytest --junitxml=result-unit.xml test\\unit
                           '''
                        }
                    }
                }
                
                stage('Service') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            bat '''
                                set FLASK_APP=app\\api.py
                                start %pathscript%\\flask run
                                start java -jar %pathwiremock% --port 9090 --root-dir %WORKSPACE%\\test\\wiremock
                                set PYTHONPATH=%WORKSPACE%
                                %pathscript%\\pytest --junitxml=result-rest.xml test\\rest
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
}
