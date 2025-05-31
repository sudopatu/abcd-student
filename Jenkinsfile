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
                    git credentialsId: 'github-pat', url: 'https://github.com/sudopatu/abcd-student', branch: 'main'
                }
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }
        stage('[ZAP] Baseline passive-scan') {
           steps {
               sh 'mkdir -p results/'
               sh '''
                   echo 'Run juice-shop...'
                   docker run --name juice-shop -d --rm \
                       -p 3000:3000 \
                       bkimminich/juice-shop
                   sleep 5
               '''
               sh '''
                   echo 'Run zap...'
                   docker run --user root --name zap \
                       --add-host=host.docker.internal:host-gateway \
                       -v /mnt/c/szkolenia/abcdevsecops/abcd-student/.zap:/zap/wrk/:rw \
                       -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                       "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                       || true
                       docker stop zap
               '''
           }
           post {
               always {
                   sh '''
                       echo 'Archiving results and stop...'
                       docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                       docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                       docker stop zap juice-shop
                    '''
                    archiveArtifacts artifacts: '${WORKSPACE}/results/zap_html_report.html', fingerprint: true, allowEmptyArchive: true
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                }
           }
        }
        stage('OSV scan') {
            steps {
                sh 'mkdir -p results/'
                sh '''
                    echo 'Run juice-shop...'
                    ls -la results/
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json 
                '''
            }
            post {
                always {
                    sh '''
                        echo 'Archiving results and stop...'
                        ls -la ${WORKSPACE}
                        ls -la ${WORKSPACE}/results/
                        docker stop juice-shop
                    '''
                    archiveArtifacts artifacts: '${WORKSPACE}/results/sca-osv-scanner.json', fingerprint: true, allowEmptyArchive: true
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                }
            }
        }
        stage('Trufflehog scan') {
            steps {
                sh 'rm -rf mirror_repo/'
                sh 'mkdir -p mirror_repo/'
                sh '''
                    echo 'Run trufflehog...'

                    git clone --mirror --branch main 'https://github.com/sudopatu/abcd-student' mirror_repo
                    cd mirror_repo/
                    ls -la
                    pwd
                    trufflehog git file://. --branch main --only-verified --bare  --json > raport.json
                '''

            }
            post {
                always {
                    sh '''
                        echo 'Archiving results...'
                    '''
                    archiveArtifacts artifacts: 'mirror_repo/raport.json', fingerprint: true, allowEmptyArchive: true
                    // archiveArtifacts artifacts: 'mirror_repo/**/*', fingerprint: true, allowEmptyArchive: true
                }
            }
        }
        stage('Semgrep scan') {
            steps {
                sh 'rm -rf semgrep_results/'
                sh 'mkdir -p semgrep_results/'
                sh '''
                    echo 'Run semogrep...'
                    semgrep scan --config auto --json --output semgrep_results/semgrep.json
                    ls -la semgrep_results/
                '''
            }
            post {
                always {
                    sh '''
                        echo 'Archiving results...'
                    '''
                    archiveArtifacts artifacts: 'semgrep_results/semgrep.json', fingerprint: true, allowEmptyArchive: true
                }
            }
        }
    }
}