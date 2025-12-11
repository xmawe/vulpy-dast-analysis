pipeline {
    agent any
    
    environment {
        ZAP_HOST = 'owasp-zap'
        ZAP_PORT = '8090'
        ZAP_API_KEY = 'dast-api-key-2025'
        VULPY_BAD_URL = 'http://vulpy-bad:5000'
        VULPY_GOOD_URL = 'http://vulpy-good:5000'
        REPORTS_DIR = "${WORKSPACE}/dast-reports"
    }
    
    stages {
        stage('Setup') {
            steps {
                sh '''
                    mkdir -p ${REPORTS_DIR}
                    echo "DAST Analysis Started: $(date)" > ${REPORTS_DIR}/scan-info.txt
                    echo "ZAP Host: ${ZAP_HOST}:${ZAP_PORT}" >> ${REPORTS_DIR}/scan-info.txt
                '''
            }
        }
        
        stage('Wait for Services') {
            steps {
                sh '''
                    echo "Waiting for services to be ready..."
                    
                    # Wait for ZAP
                    for i in {1..30}; do
                        if curl -s http://${ZAP_HOST}:${ZAP_PORT} > /dev/null 2>&1; then
                            echo "ZAP is ready"
                            break
                        fi
                        echo "Waiting for ZAP... ($i/30)"
                        sleep 2
                    done
                    
                    # Wait for Vulpy Bad
                    for i in {1..30}; do
                        if curl -s ${VULPY_BAD_URL} > /dev/null 2>&1; then
                            echo "Vulpy Bad is ready"
                            break
                        fi
                        echo "Waiting for Vulpy Bad... ($i/30)"
                        sleep 2
                    done
                    
                    # Wait for Vulpy Good
                    for i in {1..30}; do
                        if curl -s ${VULPY_GOOD_URL} > /dev/null 2>&1; then
                            echo "Vulpy Good is ready"
                            break
                        fi
                        echo "Waiting for Vulpy Good... ($i/30)"
                        sleep 2
                    done
                '''
            }
        }
        
        // ========== DAST Scan - Bad (Vulnerable) Version ==========
        stage('Spider Scan - Bad Version') {
            steps {
                sh '''
                    echo "Starting spider scan on Bad version..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/JSON/spider/action/scan/?apikey=${ZAP_API_KEY}&url=${VULPY_BAD_URL}&maxChildren=10&recurse=true&contextName=&subtreeOnly="
                    
                    # Wait for spider to complete
                    sleep 10
                '''
            }
        }
        
        stage('Active Scan - Bad Version') {
            steps {
                sh '''
                    echo "Starting active scan on Bad version..."
                    SCAN_ID=$(curl -s "http://${ZAP_HOST}:${ZAP_PORT}/JSON/ascan/action/scan/?apikey=${ZAP_API_KEY}&url=${VULPY_BAD_URL}&recurse=true&inScopeOnly=&scanPolicyName=&method=&postData=&contextId=" | jq -r '.scan')
                    
                    echo "Scan ID: $SCAN_ID"
                    
                    if [ "$SCAN_ID" = "null" ] || [ -z "$SCAN_ID" ]; then
                        echo "ERROR: Failed to start scan. Check if target is accessible."
                        exit 1
                    fi
                    
                    # Wait for active scan to complete (max 20 minutes)
                    TIMEOUT=240
                    ELAPSED=0
                    while true; do
                        STATUS=$(curl -s "http://${ZAP_HOST}:${ZAP_PORT}/JSON/ascan/view/status/?apikey=${ZAP_API_KEY}&scanId=$SCAN_ID" | jq -r '.status')
                        echo "Scan progress: $STATUS%"
                        
                        if [ "$STATUS" = "100" ]; then
                            echo "Scan completed!"
                            break
                        fi
                        
                        if [ $ELAPSED -ge $TIMEOUT ]; then
                            echo "ERROR: Scan timeout after $TIMEOUT iterations"
                            exit 1
                        fi
                        
                        sleep 5
                        ELAPSED=$((ELAPSED + 1))
                    done
                '''
            }
        }
        
        stage('Generate Report - Bad Version') {
            steps {
                sh '''
                    echo "Generating HTML report for Bad version..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/OTHER/core/other/htmlreport/?apikey=${ZAP_API_KEY}" -o ${REPORTS_DIR}/zap-report-bad.html
                    
                    echo "Generating JSON report for Bad version..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/JSON/core/view/alerts/?apikey=${ZAP_API_KEY}&baseurl=${VULPY_BAD_URL}" -o ${REPORTS_DIR}/zap-alerts-bad.json
                    
                    echo "Generating XML report for Bad version..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/OTHER/core/other/xmlreport/?apikey=${ZAP_API_KEY}" -o ${REPORTS_DIR}/zap-report-bad.xml
                '''
            }
        }
        
        // ========== DAST Scan - Good (Secure) Version ==========
        stage('Spider Scan - Good Version') {
            steps {
                sh '''
                    echo "Starting spider scan on Good version..."
                    
                    # Clear previous session
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/JSON/core/action/newSession/?apikey=${ZAP_API_KEY}&name=good-scan&overwrite=true"
                    
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/JSON/spider/action/scan/?apikey=${ZAP_API_KEY}&url=${VULPY_GOOD_URL}&maxChildren=10&recurse=true&contextName=&subtreeOnly="
                    
                    sleep 10
                '''
            }
        }
        
        stage('Active Scan - Good Version') {
            steps {
                sh '''
                    echo "Starting active scan on Good version..."
                    SCAN_ID=$(curl -s "http://${ZAP_HOST}:${ZAP_PORT}/JSON/ascan/action/scan/?apikey=${ZAP_API_KEY}&url=${VULPY_GOOD_URL}&recurse=true&inScopeOnly=&scanPolicyName=&method=&postData=&contextId=" | jq -r '.scan')
                    
                    echo "Scan ID: $SCAN_ID"
                    
                    if [ "$SCAN_ID" = "null" ] || [ -z "$SCAN_ID" ]; then
                        echo "ERROR: Failed to start scan. Check if target is accessible."
                        exit 1
                    fi
                    
                    # Wait for active scan to complete (max 20 minutes)
                    TIMEOUT=240
                    ELAPSED=0
                    while true; do
                        STATUS=$(curl -s "http://${ZAP_HOST}:${ZAP_PORT}/JSON/ascan/view/status/?apikey=${ZAP_API_KEY}&scanId=$SCAN_ID" | jq -r '.status')
                        echo "Scan progress: $STATUS%"
                        
                        if [ "$STATUS" = "100" ]; then
                            echo "Scan completed!"
                            break
                        fi
                        
                        if [ $ELAPSED -ge $TIMEOUT ]; then
                            echo "ERROR: Scan timeout after $TIMEOUT iterations"
                            exit 1
                        fi
                        
                        sleep 5
                        ELAPSED=$((ELAPSED + 1))
                    done
                '''
            }
        }
        
        stage('Generate Report - Good Version') {
            steps {
                sh '''
                    echo "Generating HTML report for Good version..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/OTHER/core/other/htmlreport/?apikey=${ZAP_API_KEY}" -o ${REPORTS_DIR}/zap-report-good.html
                    
                    echo "Generating JSON report for Good version..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/JSON/core/view/alerts/?apikey=${ZAP_API_KEY}&baseurl=${VULPY_GOOD_URL}" -o ${REPORTS_DIR}/zap-alerts-good.json
                    
                    echo "Generating XML report for Good version..."
                    curl "http://${ZAP_HOST}:${ZAP_PORT}/OTHER/core/other/xmlreport/?apikey=${ZAP_API_KEY}" -o ${REPORTS_DIR}/zap-report-good.xml
                '''
            }
        }
        
        stage('Summary') {
            steps {
                sh '''
                    echo "=== DAST Scan Summary ===" | tee -a ${REPORTS_DIR}/scan-info.txt
                    echo "Reports generated:" | tee -a ${REPORTS_DIR}/scan-info.txt
                    ls -lh ${REPORTS_DIR}/ | tee -a ${REPORTS_DIR}/scan-info.txt
                    
                    # Count alerts by risk level for Bad version
                    echo "" | tee -a ${REPORTS_DIR}/scan-info.txt
                    echo "Bad Version Alert Summary:" | tee -a ${REPORTS_DIR}/scan-info.txt
                    if [ -f ${REPORTS_DIR}/zap-alerts-bad.json ]; then
                        HIGH=$(cat ${REPORTS_DIR}/zap-alerts-bad.json | jq '[.alerts[] | select(.risk=="High")] | length')
                        MEDIUM=$(cat ${REPORTS_DIR}/zap-alerts-bad.json | jq '[.alerts[] | select(.risk=="Medium")] | length')
                        LOW=$(cat ${REPORTS_DIR}/zap-alerts-bad.json | jq '[.alerts[] | select(.risk=="Low")] | length')
                        INFO=$(cat ${REPORTS_DIR}/zap-alerts-bad.json | jq '[.alerts[] | select(.risk=="Informational")] | length')
                        
                        echo "  High: $HIGH" | tee -a ${REPORTS_DIR}/scan-info.txt
                        echo "  Medium: $MEDIUM" | tee -a ${REPORTS_DIR}/scan-info.txt
                        echo "  Low: $LOW" | tee -a ${REPORTS_DIR}/scan-info.txt
                        echo "  Informational: $INFO" | tee -a ${REPORTS_DIR}/scan-info.txt
                    fi
                    
                    # Count alerts by risk level for Good version
                    echo "" | tee -a ${REPORTS_DIR}/scan-info.txt
                    echo "Good Version Alert Summary:" | tee -a ${REPORTS_DIR}/scan-info.txt
                    if [ -f ${REPORTS_DIR}/zap-alerts-good.json ]; then
                        HIGH=$(cat ${REPORTS_DIR}/zap-alerts-good.json | jq '[.alerts[] | select(.risk=="High")] | length')
                        MEDIUM=$(cat ${REPORTS_DIR}/zap-alerts-good.json | jq '[.alerts[] | select(.risk=="Medium")] | length')
                        LOW=$(cat ${REPORTS_DIR}/zap-alerts-good.json | jq '[.alerts[] | select(.risk=="Low")] | length')
                        INFO=$(cat ${REPORTS_DIR}/zap-alerts-good.json | jq '[.alerts[] | select(.risk=="Informational")] | length')
                        
                        echo "  High: $HIGH" | tee -a ${REPORTS_DIR}/scan-info.txt
                        echo "  Medium: $MEDIUM" | tee -a ${REPORTS_DIR}/scan-info.txt
                        echo "  Low: $LOW" | tee -a ${REPORTS_DIR}/scan-info.txt
                        echo "  Informational: $INFO" | tee -a ${REPORTS_DIR}/scan-info.txt
                    fi
                '''
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'dast-reports/**', allowEmptyArchive: true
            
            publishHTML([
                reportDir: 'dast-reports',
                reportFiles: 'zap-report-bad.html, zap-report-good.html',
                reportName: 'DAST Security Reports',
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true
            ])
        }
        success {
            echo 'DAST Analysis completed successfully!'
        }
        failure {
            echo 'DAST Analysis failed. Check logs for details.'
        }
    }
}
