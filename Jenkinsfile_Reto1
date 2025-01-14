pipeline { 

    agent any 

    stages { 

        stage('Get Code') { 

            steps { 

                git 'https://github.com/rfbm97/Modulo2' 

            } 

        } 

        stage('Build') { 

            steps { 

                script { 

                    echo "Workspace directory: ${env.WORKSPACE}" 

                } 

            } 

        } 

        stage ('Tests'){ 

            parallel{ 

                stage('Unit Tests') { 

                    steps { 

                        sh ''' 

                        export PYTHONPATH=${WORKSPACE} 

                        pytest --junitxml=result-unit.xml ${WORKSPACE}/test/unit 

                        ''' 

                    } 

                } 

                stage('Services') { 

                    steps { 

                        sh ''' 

                            export FLASK_APP=${WORKSPACE}/app/api.py 

                            flask run --host=0.0.0.0 --port=5000 & 

                            export PYTHONPATH=${WORKSPACE} 

                            docker pull wiremock/wiremock:latest 

                            docker run -d --rm -p 9090:8080 -v ${WORKSPACE}/test/wiremock/mappings:/home/wiremock/mappings wiremock/wiremock:latest & 

                            pytest --junitxml=result-unit.xml ${WORKSPACE}/test/rest 

                        ''' 

                    } 

                } 

            } 

        } 

             

         

        stage('Results'){ 

            steps{ 

                junit 'result-unit.xml' 

            } 

        } 

    } 

}  
