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
                    git credentialsId: 'github-pat', url: 'https://github.com/psiewert999/abcd-student', branch: 'main'
                }
            }
        }
        stage('Getting ready') {
            steps {
                sh 'mkdir -p wyniki'
            }
        }
        stage('[ZAP] passive-scan') {
            steps {
                sh '''
                docker run --name juice-shop -d --rm \
                -p 3000:3000 \
                bkimminich/juice-shop
                sleep 6
                '''
                sh '''
                docker run --name zap \
            	-v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/.zap:/zap/wrk/:rw \
                -t ghcr.io/zaproxy/zaproxy:stable \
                bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                || true
                '''
            }
            post {
                always {
                    sh '''
                    docker cp zap:/zap/wrk/reports/zap_html_report.html ${PWD}/wyniki/zap_html_report.html
                    docker cp zap:/zap/wrk/reports/zap_html_report.xml ${PWD}/wyniki/zap_html_report.xml
                    docker stop juice-shop
                    docker rm zap       	    
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'archiwizacja wynikow'
            archiveArtifacts artifacs: 'wyniki/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo'
        }
    }
}