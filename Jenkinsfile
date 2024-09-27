pipeline {
    agent any
    tools {
        nodejs 'Node_18' // Use a stable NodeJS version
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/oleksandr-bondarenko-aqa/docker-jenkins-excersise.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
                sh 'npm install playwright mocha @reportportal/agent-js-mocha@5.0.3'
                sh 'chmod +x ./node_modules/.bin/*'
                sh 'npx playwright install-deps'
                sh 'npx playwright install'
            }
        }
        stage('Configure ReportPortal') {
            steps {
                writeFile file: 'reportportal.conf.json', text: '''{
  "endpoint": "http://192.168.0.108:8081/api/v1",
  "project": "superadmin_personal",
  "launch": "Playwright Test Run",
  "description": "Playwright tests",
  "attributes": [
    {
      "key": "platform",
      "value": "jenkins"
    }
  ],
  "mode": "DEFAULT",
  "debug": true
}'''
            }
        }
        stage('Verify Configuration') {
            steps {
                sh 'cat reportportal.conf.json'
            }
        }
        stage('Run Tests') {
            steps {
                withCredentials([string(credentialsId: 'reportportal-token', variable: 'RP_TOKEN')]) {
                    sh 'echo "RP_TOKEN is set"'
                    // Print non-sensitive environment variables
                    sh 'env | grep RP_ | grep -v RP_TOKEN || true'
                    // Run the tests with set -e
                    sh '''
                    set -e
                    npx mocha --reporter @reportportal/agent-js-mocha --reporter-options \
                    "configFile=./reportportal.conf.json,token=${RP_TOKEN}" test/loginTest.js
                    '''
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                color: '#36a64f',
                message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' succeeded.\nReportPortal: http://192.168.0.108:8081/ui/#superadmin_personal/launches/all\nSee details: ${env.BUILD_URL}"
            )
        }
        failure {
            slackSend(
                color: '#FF0000',
                message: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed.\nReportPortal: http://192.168.0.108:8081/ui/#superadmin_personal/launches/all\nSee details: ${env.BUILD_URL}"
            )
        }
    }
}
