pipeline {
    agent none

    stages {
        //Añadimos etapa de limpieza de workspace para asegurarnos que no hay ni archivos bloqueados ni problemas de permisos
        stage('Clean Workspace'){
            agent {label 'git'}
            steps {
                deleteDir()
                echo "Workspace cleaned"
            }
        }
        stage('Get Code') {
            agent {label 'git'}
            steps {

                echo 'Usuario en el que se ejecuta la sesion'
                sh 'whoami'

                echo 'hostname:'
                sh 'hostname'

                //Descargamos el codigo del repositorio
                git 'https://github.com/rfbm97/Modulo2'
                // Guardo los archivos en un área de almacenamiento temporal compartido entre los agentes con stash 
                stash includes: '**/*', name: 'Files'
                
            }
        }
        stage('Build') {
            agent {label 'git'}
            steps {
                //Nos aseguramos cuál es nuestro workspace
                script {
                    echo "Workspace directory: ${env.WORKSPACE}"
                }
                echo 'Usuario en el que se ejecuta la sesion'
                sh 'whoami'

                echo 'hostname:'
                sh 'hostname'
            }
        }
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent {label 'pytest'}
                    steps {
                        echo 'Usuario en el que se ejecuta la sesion'
                        sh 'whoami'

                        echo 'hostname:'
                        sh 'hostname'
                
                        //Cojo los archivos del area de almacenamiento temporal generada con stash
                        unstash 'Files'
                        //Lanzamos pruebas unitarias
                        sh '''
                        export PYTHONPATH=${WORKSPACE}
                        pytest --junitxml=result-unit.xml ${WORKSPACE}/test/unit
                        sleep 11 #Aseguramos que wiremock y flask estan activados
                        pytest --junitxml=result-rest.xml ${WORKSPACE}/test/rest
                        '''
                        
                    }
                }
                stage ('Wiremock Service') {
                    //Lanzamos wiremock en un nodo diferente
                    agent {label 'wiremock'}
                    steps {
                        echo 'Usuario en el que se ejecuta la sesion'
                        sh 'whoami'

                        echo 'hostname:'
                        sh 'hostname'
                        unstash 'Files'
                        sh '''
                            export PYTHONPATH=${WORKSPACE}
                            docker pull wiremock/wiremock:latest
                            docker run -d --rm -p 9090:8080 -v ${WORKSPACE}/test/wiremock/mappings:/home/wiremock/mappings wiremock/wiremock:latest &
                            sleep 10 # Aseguramos que Wiremock esté disponible 
                        '''
                    }
                }
                stage('Flask Service') {
                    //Lanzamos flask en un nodo diferente
                    agent {label 'flask'}
                    steps {
                        echo 'Usuario en el que se ejecuta la sesion'
                        sh 'whoami'

                        echo 'hostname:'
                        sh 'hostname'
                        unstash 'Files'
                        
                        script {
                            echo "Workspace directory: ${env.WORKSPACE}"
                        }
                        sh '''
                            export FLASK_APP=${WORKSPACE}/app/api.py
                            flask run --host=0.0.0.0 --port=5000 &
                            export PYTHONPATH=${WORKSPACE}
                            sleep 10 #Aseguramos que flask esté disponible
                        '''
                    }
                }
            }
        }
        stage('Results') {
            agent {label 'pytest'}
            steps {
                echo 'Usuario en el que se ejecuta la sesion'
                sh 'whoami'

                echo 'hostname:'
                sh 'hostname'
                
                junit 'result-unit.xml'
                junit 'result-rest.xml'
            }
        }
    }
}
