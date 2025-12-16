pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '10'))
    }

    environment {
        PYTHON_VERSION = '3.x'
        TEST_RESULTS_DIR = 'test-results'
        REPORTS_DIR = 'html-reports'
        VENV_DIR = '.venv'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo '========== Checking out source code =========='
                    checkout scm
                }
            }
        }

        stage('Setup Python Environment') {
            steps {
                script {
                    echo '========== Setting up Python virtual environment =========='
                    sh '''
                        python3 -m venv ${VENV_DIR}
                        . ${VENV_DIR}/bin/activate
                        python --version
                        pip install --upgrade pip setuptools wheel
                    '''
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    echo '========== Installing project dependencies =========='
                    sh '''
                        . ${VENV_DIR}/bin/activate
                        
                        # Install requirements if requirements.txt exists
                        if [ -f requirements.txt ]; then
                            pip install -r requirements.txt
                        fi
                        
                        # Install testing dependencies
                        pip install pytest pytest-html pytest-cov pytest-json-report
                    '''
                }
            }
        }

        stage('Create Test Directories') {
            steps {
                script {
                    echo '========== Creating test and report directories =========='
                    sh '''
                        mkdir -p ${TEST_RESULTS_DIR}
                        mkdir -p ${REPORTS_DIR}
                    '''
                }
            }
        }

        stage('Run Tests with Coverage') {
            steps {
                script {
                    echo '========== Running pytest tests with coverage =========='
                    sh '''
                        . ${VENV_DIR}/bin/activate
                        
                        # Run pytest with HTML report, coverage, and JSON report
                        pytest \
                            --verbose \
                            --tb=short \
                            --junit-xml=${TEST_RESULTS_DIR}/junit-report.xml \
                            --html=${REPORTS_DIR}/pytest-report.html \
                            --self-contained-html \
                            --cov=. \
                            --cov-report=html:${REPORTS_DIR}/coverage-report \
                            --cov-report=term-missing \
                            --json-report \
                            --json-report-file=${TEST_RESULTS_DIR}/report.json \
                            tests/ || true
                    '''
                }
            }
        }

        stage('Generate HTML Test Report') {
            steps {
                script {
                    echo '========== Generating comprehensive HTML test report =========='
                    sh '''
                        . ${VENV_DIR}/bin/activate
                        
                        # Create a summary HTML report
                        cat > ${REPORTS_DIR}/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Test Reports Dashboard</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            border-bottom: 3px solid #007bff;
            padding-bottom: 10px;
        }
        .report-section {
            margin: 20px 0;
            padding: 15px;
            border-left: 4px solid #007bff;
            background-color: #f9f9f9;
        }
        .report-section h2 {
            color: #007bff;
            margin-top: 0;
        }
        .report-link {
            display: inline-block;
            padding: 10px 20px;
            margin: 5px;
            background-color: #007bff;
            color: white;
            text-decoration: none;
            border-radius: 4px;
            transition: background-color 0.3s;
        }
        .report-link:hover {
            background-color: #0056b3;
        }
        .info {
            color: #666;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🧪 Test Reports Dashboard</h1>
        
        <div class="report-section">
            <h2>📊 Available Reports</h2>
            <p class="info">Click on any report below to view detailed test results and metrics.</p>
            <a href="pytest-report.html" class="report-link">📋 Pytest Test Report</a>
            <a href="coverage-report/index.html" class="report-link">📈 Code Coverage Report</a>
        </div>
        
        <div class="report-section">
            <h2>ℹ️ Report Information</h2>
            <p><strong>Generated:</strong> <span id="timestamp"></span></p>
            <p><strong>Build URL:</strong> ${BUILD_URL}</p>
            <p><strong>Build Number:</strong> ${BUILD_NUMBER}</p>
        </div>
    </div>
    
    <script>
        document.getElementById('timestamp').textContent = new Date().toLocaleString();
    </script>
</body>
</html>
EOF
                    '''
                }
            }
        }

        stage('Archive Reports') {
            steps {
                script {
                    echo '========== Archiving test reports and artifacts =========='
                    archiveArtifacts artifacts: "${REPORTS_DIR}/**,${TEST_RESULTS_DIR}/**", 
                                     allowEmptyArchive: true
                }
            }
        }

        stage('Publish Test Results') {
            steps {
                script {
                    echo '========== Publishing test results =========='
                    junit testResults: "${TEST_RESULTS_DIR}/junit-report.xml", 
                          allowEmptyResults: true
                }
            }
        }
    }

    post {
        always {
            script {
                echo '========== Pipeline Cleanup =========='
                // Clean workspace if needed
                cleanWs(deleteDirs: true, patterns: [[pattern: '${VENV_DIR}', type: 'INCLUDE']])
            }
        }
        success {
            script {
                echo '========== Pipeline Successful =========='
                echo "✅ All tests passed! Reports are available at: ${BUILD_URL}artifact/${REPORTS_DIR}/index.html"
            }
        }
        failure {
            script {
                echo '========== Pipeline Failed =========='
                echo "❌ Some tests failed. Check the reports for details."
            }
        }
        unstable {
            script {
                echo '========== Pipeline Unstable =========='
                echo "⚠️ Pipeline is unstable. Review the test results."
            }
        }
    }
}
