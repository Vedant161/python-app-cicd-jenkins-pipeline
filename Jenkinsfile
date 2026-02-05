pipeline {
    agent any

    environment {
        VENV = "venv"
        PYTHON = "python3"
    }

    stages {

        stage('Clean Reports') {
            steps {
                sh '''
                echo "********* Cleaning Workspace Stage Started **********"
                rm -rf test-reports dist build
                mkdir -p test-reports
                echo "********* Cleaning Workspace Stage Finished **********"
                '''
            }
        }

        stage('Setup Environment') {
            steps {
                sh """
                ${PYTHON} -m venv ${VENV}
                . ${VENV}/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                pip install pytest pytest-html pytest-cov pyinstaller
                """
            }
        }

        stage('Build Stage') {
            steps {
                sh """
                echo '********* Build Stage Started **********'
                . ${VENV}/bin/activate
                pyinstaller --onefile app.py
                echo '********* Build Stage Finished **********'
                """
            }
        }

        stage('Testing Stage') {
            steps {
                sh """
                echo '********* Test Stage Started **********'
                . ${VENV}/bin/activate
                pytest test.py \
                  --junitxml=test-reports/results.xml \
                  --html=test-reports/report.html \
                  --self-contained-html
                echo '********* Test Stage Finished **********'
                """
            }
        }

        stage('Sanity Approval') {
            steps {
                input message: "Does the staging environment look OK?"
            }
        }

        stage('Deployment Approval') {
            steps {
                input message: "Do you want to deploy the application?"
            }
        }

        stage('Deployment Stage') {
            steps {
                echo '********* Deploy Stage Started **********'
                sh "echo 'Deploying build artifact... (replace with real deploy command)'"
                echo '********* Deploy Stage Finished **********'
            }
        }
    }

    post {
        always {
            echo 'We came to an end!'

            archiveArtifacts artifacts: 'dist/*', fingerprint: true

            junit 'test-reports/results.xml'

            publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'test-reports',
                reportFiles: 'report.html',
                reportName: 'Pytest HTML Report'
            ])

            deleteDir()
        }

        success {
            echo 'Build Successful!! 🎉'
        }

        failure {
            echo 'Build Failed. Check logs & test reports.'
        }

        unstable {
            echo 'Run was marked as UNSTABLE'
        }

        changed {
            echo 'Pipeline state has changed.'
        }
    }
}
