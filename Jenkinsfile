pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        // stage('Code checkout from GitHub') {
        //     steps {
        //         script {
        //             cleanWs()
        //             git credentialsId: 'github-pat', url: 'https://github.com/sudopatu/abcd-student', branch: 'main'
        //         }
        //     }
        // }
        // stage('Example') {
        //     steps {
        //         echo 'Hello!'
        //         sh 'ls -la'
        //     }
        // }
        //stage('[ZAP] Baseline passive-scan') {
        //    steps {
        //        sh 'mkdir -p results/'
        //        sh '''
        //            echo 'Run juice-shop...'
        //            docker run --name juice-shop -d --rm \
        //                -p 3000:3000 \
        //                bkimminich/juice-shop
        //            sleep 5
        //        '''
        //        sh '''
        //            echo 'Run zap...'
        //            docker run --user root --name zap \
        //                --add-host=host.docker.internal:host-gateway \
        //                -v /mnt/c/szkolenia/abcdevsecops/abcd-student/.zap:/zap/wrk/:rw \
        //                -t ghcr.io/zaproxy/zaproxy:stable bash -c \
        //                "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
        //                || true
        //        '''
        //    }
        //    post {
        //        always {
        //            sh '''
        //                echo 'Archiving results and stop...'
        //                docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
        //                docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
        //                docker stop zap juice-shop
        //                docker rm zap
        //            '''
        //        }
        //    }
        //}
        stage('OSV scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    echo 'Run juice-shop...'
                    ls -la results/
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json &
                    echo "osv-scanner exited with code $?" &
                    pid=$!
                    echo "Scan completed" 
                    echo "Waiting for osv-scanner process (PID $pid) to finish..."
                    wait $pid
                    cat results/sca-osv-scanner.json & ls -la results/
                    wait $!
                    cat results/sca-osv-scanner.json
                '''
            }
            post {
                always {
                    sh '''
                        echo 'Archiving results and stop...'
                        archiveArtifacts artifacts: 'results/sca-osv-scanner.json', fingerprint: true, allowEmptyArchive: true
                        docker stop juice-shop
                    '''
                }
            }
        }
    }
}