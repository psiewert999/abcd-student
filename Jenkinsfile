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
                script {
                                        sh 'mkdir -p wyniki'
                    def isJuiceShopRunning = sh(script: "docker ps -a --filter 'name=juice-shop' --filter 'status=running' -q", returnStdout: true).trim()
                    def isZapRunning = sh(script: "docker ps -a --filter 'name=zap2' --filter 'status=running' -q", returnStdout: true).trim()
                    sh '''
                    docker ps
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
                        sh '''
                        docker ps -a --filter "exited=0" -q | xargs docker rmd
                        '''
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
                    docker run --name zap2 \
                    -v /home/psiewert/KURS_ABC_DEVSECOPS/abcd-student/.zap:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable \
                    bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                '''
            }
            post {
                always {
                    sh '''
                        docker exec zap2 pwd
                        docker exec zap2 ls
                        docker stop juice-shop
                        docker stop zap2                            	    
                    '''
                }
            }
        }
    }
    post {
        always {
            sh'''
            docker ps
            '''
            echo 'archiwizacja wynikow'
            archiveArtifacts artifacts: 'wyniki/**/*', fingerprint: true, allowEmptyArchive: true
            echo 'Sending reports to DefectDojo'
            }
    }
}