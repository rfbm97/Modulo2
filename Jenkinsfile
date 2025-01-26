pipeline { 

    agent any 

    stages { 

        stage('Get Code') { 

            steps { 

                git branch: 'develop', url: 'https://github.com/rfbm97/Modulo2' 

            } 

        } 

        stage('Unit') { 

            steps { 

                sh ''' 
                export PYTHONPATH=${WORKSPACE} 

                pytest --junitxml=result-unit.xml ${WORKSPACE}/test/unit 

                ''' 

                junit 'result-unit.xml' 

            } 

        } 

        stage('Rest') { 

            steps { 

                sh ''' 
                    # Lanzamos Flask
                    export FLASK_APP=${WORKSPACE}/app/api.py 

                    flask run --host=0.0.0.0 --port=5000 & 

                    export PYTHONPATH=${WORKSPACE} 

                    # Lanzamos Wiremock

                    docker pull wiremock/wiremock:latest 

                    docker run -d --rm -p 9090:8080 -v ${WORKSPACE}/test/wiremock/mappings:/home/wiremock/mappings wiremock/wiremock:latest & 

                    sleep 11

                    # Ejecutamos las pruebas de integración
                    pytest --junitxml=result-unit.xml ${WORKSPACE}/test/rest 

                ''' 

            } 

        } 

        stage('Static'){
            steps{
                // Ejecutamos flake8 para realizar las pruebas estáticas y exportamos los resultados a flake8.out
                sh '''
                python3 -m flake8 --format=pylint --exit-zero app>flake8.out
                '''
                // Vemos los resultados de forma gráfica, utilizando el plugin warnings-ng
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                }
            }
        }

        stage('Security'){
            steps{
                // Ejecutamos bandit para realizar las pruebas de seguridad
                sh'''
                python3 -m bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"
                '''
                // Vemos los resultados de forma gráfica, utilizando el plugin warnings-ng
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
                }
            }

        }

        stage('Performance'){
            steps{
                sh '''

                    # Verificamos que Flask está levantado
                    until curl -s http://localhost:5000; do echo "Esperando a Flask..."; sleep 5; done
                   
                    # Realizamos pruebas de rendimiento
                    /usr/local/bin/apache-jmeter-5.6.3/bin/jmeter -n -t ${WORKSPACE}/test/jmeter/flask.jmx -f -l flask.jtl
                '''

                // Ejecutamos el plugin performance para la visualización de los resultados
                perfReport sourceDataFiles: 'flask.jtl'

            }
       }
        
        stage('Coverage'){
            steps{
                
                sh '''
                # Lanzamos pruebas de cobertura
                python3 -m coverage run --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                
                # Exportamos los resultados
                python3 -m coverage xml
                '''

                // Exponemos los resultados de forma gráfica con el plugin cobertura
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){    
                    cobertura coberturaReportFile: 'coverage.xml', onlyStable: false, conditionalCoverageTargets: '90,0,80', lineCoverageTargets: '95,0,85'
                }
            }
        }

    } 

}
