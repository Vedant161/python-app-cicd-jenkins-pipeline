pipeline {
    agent any

    environment {
        CONDA_DIR = "${WORKSPACE}/miniconda"
        PATH = "${CONDA_DIR}/bin:${env.PATH}"
    }

    stages {

        stage('Setup Python (Local)') {
            steps {
                sh '''
                echo "Downloading Miniconda..."
                wget -q https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh
                bash miniconda.sh -b -p $CONDA_DIR
                conda init bash
                conda --version
                '''
            }
        }

        stage('Create Virtual Environment') {
            steps {
                sh '''
                conda create -y -n ci-env python=3.11
                source $CONDA_DIR/bin/activate ci-env
                python --version
                pip install --upgrade pip
                pip install -r requirements.txt
                pip install pytest pytest-html pytest-cov pyinstaller
                mkdir -p test-reports
                '''
            }
        }

        stage('Build / Compile Check') {
            steps {
                sh '''
                source $CONDA_DIR/bin/activate ci-env
                python -m py_compile app.py
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                source $CONDA_DIR/bin/activate ci-env
                pytest test.py \
                  --junitxml=test-reports/results.xml \
                  --html=test-reports/report.html \
                  --self-contained-html
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'test-reports/results.xml'
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'test-reports',
                        reportFiles: 'report.html',
                        reportName: 'Pytest HTML Report'
                    ])
                }
            }
        }
    }

    post {
        success {
            echo 'Python CI Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
