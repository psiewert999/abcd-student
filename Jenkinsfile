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
                    sh 'docker start juice-shop || docker run --name juice-shop -d --rm -p 172.17.0.1:3000:3000 -p 127.0.0.1:3000:3000 bkimminich/juice-shop'
				    }
            }
        }
        stage('[SCA] supply-chain') {
            steps {
                sh '''
                    docker run -v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/:/app \
                    -t osv-scanner scan /app/package-lock.json \
                    -v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/reports:/app:rw \
                    --format table --output /reports/osv-scan_report.json -t osv-scanner || true
                '''
                sh '''

                    docker run -v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/:/app \
                    -v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/reports:/app:rw \
                    -t osv-scanner scan /app/package-lock.json \
                    --format table \ 
                    --output /reports/osv-scan_report.txt -t osv-scanner || true
                
                    '''
            }
            post {
                
                always {
2
                sh '''
                
                docker cp osv-scanner:/app/osv-report.json ${WORKSPACE}/reports/osv-report.json
                docker cp osv-scanner:/app/osv-report.txt ${WORKSPACE}/reports/osv-report.txt
                docker rm /app/osv-scanner

                '''
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
                    docker rm zap2
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
            defectDojoPublisher(artifact: 'wyniki/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'patryk.siewert@opi.org.opi.pl')
            }
    }
}