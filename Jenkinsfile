pipeline {
    agent any
    tools {
        nodejs 'Node_22'
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
                sh 'npm install playwright mocha @reportportal/agent-js-mocha'
                sh 'chmod +x ./node_modules/.bin/*'
                sh 'npx playwright install-deps'
                sh 'npx playwright install'
            }
        }
        stage('Configure ReportPortal') {
            steps {
                // Create the configuration file without the API key
                writeFile file: 'reportportal.conf.json', text: '''{
  "endpoint": "http://192.168.0.108:8081/api/v1",
  "project": "docker-jenkins-exercise",
  "launch": "Playwright Test Run",
  "description": "Playwright tests",
  "attributes": [
    {
      "key": "platform",
      "value": "jenkins"
    }
  ],
  "mode": "DEFAULT",
  "debug": false
}'''
            }
        }
        stage('Run Tests') {
            steps {
                // Inject the API key as an environment variable
                withCredentials([string(credentialsId: 'reportportal-token', variable: 'RP_TOKEN')]) {
                    sh 'npx mocha --reporter @reportportal/agent-js-mocha --reporter-options configFile=./reportportal.conf.json test/loginTest.js'
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
                message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' succeeded.\nReportPortal: http://192.168.0.108:8081/ui/#docker-jenkins-exercise/launches/all\nSee details: ${env.BUILD_URL}"
            )
        }
        failure {
            slackSend(
                color: '#FF0000',
                message: "FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed.\nReportPortal: http://192.168.0.108:8081/ui/#docker-jenkins-exercise/launches/all\nSee details: ${env.BUILD_URL}"
            )
        }
    }
}
