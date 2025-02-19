pipeline {
    agent any
    
    environment {
        pathscript = "C:\\Users\\ADM-1\\AppData\\Local\\Programs\\Python\\Python313\\Scripts"
        pathjmeter ="C:\\Users\\ADM-1\\Desktop\\apache-jmeter-5.6.3\\bin\\jmeter"
    }

    stages {
        stage('GetCode') {
            steps {
                git 'https://github.com/jlcalleja/unir1-2/'
                bat 'dir'
                echo WORKSPACE
            }
        }

        stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        %pathscript%\\coverage run --branch --source=app --omit=app\\_init_.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                    '''
                    junit 'result*.xml'
                }
            }
        }
        
        stage('Coverage') {
            steps {
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                    %pathscript%\\coverage xml
                '''
                
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100, 80, 90', failUnstable: false, onlyStable: false, lineCoverageTargets: '100, 85, 95'
                }
            }
        }

        
        stage('Static') {
            steps {
                bat '''
                    %pathscript%\\flake8 --exit-zero --format=pylint app >flake8.out
                '''
                
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                }
            }
        }
        
        stage('Security-Test') {
            steps {
                bat'''
                    %pathscript%\\bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                '''
                
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]] 
                }
            }
        }
        
        stage('Performance') {
            steps {
                bat '''
                    set FLASK_APP=app\\api.py
                    start %pathscript%\\flask run
                    %pathjmeter% -n -t test\\jmeter\\flask.jmx -f -l flask.jtl
                '''
                perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
    
    post {
        always {
            cleanWs(disableDeferredWipeout: true)
        }
    }

}