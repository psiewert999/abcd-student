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
        stage('[ZAP] Baseline passive-scan') {
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
                -t ghcr.io/zaproxy/zaproxy:stable bash -c \\
                "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \\
                -d
                || true
        '''
    }
    post {
        always {
        	script{
            sh '''
                docker stop juice-shop || true
                            '''
		defectDojoPublisher(
			artifact:'/home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/.zap/reports/zap_xml_report.xml',
			productName: 'Juice Shop',
			scanType: 'ZAP Scan',
			engagementName: 'patryk.siewert@opi.org.pl')
        }
    }
}
}}}

