pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'git-hub-token', url: 'https://github.com/psiewert999/abcd-student', branch: 'main'
                }
            }
        }
        stage('Getting ready') {
            steps {
                script {
                    sh 'mkdir -p wyniki'
                    def isJuiceShopRunning = sh(script: "docker ps -a --filter 'name=juice-shop' --filter 'status=running' -q", returnStdout: true).trim()
                    def isZapRunning = sh(script: "docker ps -a --filter 'name=zap2'"|"grep zap", returnStdout: true).trim()
                    sh '''
                    docker ps -a
                    '''
                    if (isJuiceShopRunning) { 
                        echo "JUICE SHOP IS ALREADY RUNNING. Shutting down"
                        sh '''
                        docker stop juice-shop
                        sleep 6
                        docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                        '''
                        echo "New conatainer just started"
                    } else {
                        echo "JUICE SHOP is not running. Starting container"
                        sh '''
                        docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                        '''
                    }
                    if (isZapRunning) { 
                        echo "Zap IS ALREADY RUNNING. Shutting down"
                        sh "docker rm zap2" 
                        echo "Old zap has deleted"
                    } else {
                        echo "ZAP is not running."
                        }
                }
            }
        }
        stage('[ZAP] passive-scan') {
            steps {
                sh '''
                    docker run --name zap2 -d\
                    --add-host=host.docker.internal:host-gateway \
                    -v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/.zap:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                '''
            }
            post {
                always {
                    sh '''
                    docker cp zap2:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/wyniki/zap_html_report.html
                    docker cp zap2:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/wyniki/zap_xml_report.xml
                    docker stop zap2 juice-shop
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'archiwizacja wynikow'
            archiveArtifacts artifacts: 'wyniki/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo'
            }
    }
}