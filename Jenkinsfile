pipeline {
    agent none

    stages {
        stage('Clean Workspace') {
            agent { label 'git' }
            steps {
                deleteDir()
                echo "Workspace cleaned"
            }
        }
        stage('Get Code') {
            agent { label 'git' }
            steps {
                git 'https://github.com/rfbm97/Modulo2'
                // Guardo los archivos en un área de almacenamiento temporal compartido entre los agentes con stash
                stash includes: '**/*', name: 'Files'
            }
        }
        stage('Build') {
            agent { label 'git' }
            steps {
                script {
                    echo "Workspace directory: ${env.WORKSPACE}"
                }
            }
        }
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    agent { label 'pytest' }
                    steps {
                        unstash 'Files'
                        sh '''
                        export PYTHONPATH=${WORKSPACE}
                        pytest --junitxml=result-unit.xml ${WORKSPACE}/test/unit
                        '''
                    }
                }
                stage('Wiremock Service') {
                    agent { label 'wiremock' }
                    steps {
                        unstash 'Files'
                        sh '''
                        docker pull wiremock/wiremock:latest
                        docker run -d --rm -p 9090:8080 -v ${WORKSPACE}/test/wiremock/mappings:/home/wiremock/mappings wiremock/wiremock:latest
                        sleep 10
                        '''
                    }
                }
                stage('Flask Service') {
                    agent { label 'flask' }
                    steps {
                        unstash 'Files'
                        script {
                            echo "Workspace directory: ${env.WORKSPACE}"
                        }
                        sh '''
                        export FLASK_APP=${WORKSPACE}/app/api.py
                        flask run --host=0.0.0.0 --port=5000 &
                        export PYTHONPATH=${WORKSPACE}
                        sleep 10
                        '''
                    }
                }
            }
        }
        stage('Results') {
            agent { label 'pytest' }
            steps {
                junit 'result-unit.xml'
                junit 'result-rest.xml'
            }
        }
    }
}

