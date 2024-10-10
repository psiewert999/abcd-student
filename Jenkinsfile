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
        stage('Get ready') {
            steps {
                sh 'mkdir -p results/'
            }        
        }
        
        stage('[ZAP] passive-scan') {
    steps {
        sh '''
            docker run --name juice-shop -d --rm \\
                -p 3000:3000 \\
                bkimminich/juice-shop
            sleep 6
        '''
                sh '''
            docker run --name zap --rm \\
            	--add-host=host.docker.internal:host-gateway \\
                -v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/.zap:/zap/wrk/:rw \\
                -t ghcr.io/zaproxy/zaproxy:stable \
                bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \\
                -d
                || true
        '''
    }
    post {
        always {
        	
            sh '''
                docker cp zap:/zap/wrk/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                docker stop zap juice-shop || true
                            '''
		defectDojoPublisher(
			artifact:'${WORKSPACE}/results/zap_xml_report.xml',
			productName: 'Juice Shop',
			scanType: 'ZAP Scan',
			engagementName: 'patryk.siewert@opi.org.pl')
        }
    }
}
}}

