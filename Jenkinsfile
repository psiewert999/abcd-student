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
        stage('[SAST] Semgrep scan') {
            steps {
                script {
                    sh 'semgrep scan --config auto --json-output=${WORKSPACE}/wyniki/semgrep-report.json'
                    sh 'ls ${WORKSPACE}/wyniki'
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
                    docker rm zap2
                    '''
                }
            }
        }

        stage('[SCA] supply-chain') {
            steps {
                sh '''
                    docker run --name osv-json -v "/home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/:/app" \
                    -v "/home/psiewert/KURS_ABC_DEVSECOPS/reports:/reports:rw" \
                    -t osv-scanner \
                    --lockfile /app/package-lock.json \
                    --format json \
                    --output /reports/osv-scan_report.json || true
                '''
                sh '''

                    docker run --name osv-txt -v "/home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/:/app" \
                    -v "/home/psiewert/KURS_ABC_DEVSECOPS/reports:/reports:rw" \
                    -t osv-scanner \
                    --lockfile /app/package-lock.json \
                    --format table \
                    --output /reports/osv-scan_report.txt || true
                
                    '''
            }
            post {
                
                always {
                sh '''
                
                docker cp osv-json:/reports/osv-scan_report.json ${WORKSPACE}/wyniki/osv-report.json
                docker cp osv-txt:/reports/osv-scan_report.txt ${WORKSPACE}/wyniki/osv-report.txt
                docker rm osv-txt osv-json

                '''
                }
            }
        }
        stage('[SAST] truffle-hog') {
            steps {
                sh '''
                    docker run --name truffle-json \
                    --rm \
                    truffle-hog:latest \
                    github --repo https://github.com/psiewert999/abcd-student \
                    --only-verified \
                    --json > ${WORKSPACE}/wyniki/truffle-report.json || true
                '''
                sh '''
                    docker run --name truffle-txt \
                    --rm \
                    truffle-hog:latest \
                    github --repo https://github.com/psiewert999/abcd-student \
                    --only-verified > ${WORKSPACE}/wyniki/truffle-report.txt || true
                    '''
            }
        }
    }
    post {
        always {
            echo 'archiwizacja wynikow'
            archiveArtifacts artifacts: 'wyniki/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo'
            sh '''
            ls wyniki
            '''
            defectDojoPublisher(artifact: 'wyniki/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'patryk.siewert@opi.org.opi.pl')
            defectDojoPublisher(artifact: 'wyniki/osv-report.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'patryk.siewert@opi.org.opi.pl')
            defectDojoPublisher(artifact: 'wyniki/truffle-report.json', productName: 'Juice Shop', scanType: 'Trufflehog Scan', engagementName: 'patryk.siewert@opi.org.opi.pl')
            defectDojoPublisher(artifact: 'wyniki/semgrep-report.json', productName: 'Juice Shop', scanType: 'Semgrep JSON Report', engagementName: 'patryk.siewert@opi.org.opi.pl')
            }
    }
}