pipeline {
    agent any
    triggers {
        cron('@hourly')
        pollSCM('* * * * *')
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: '30'))
        disableConcurrentBuilds()
    }
    stages {
        stage ('Checkout') {
            steps {
                git url: 'https://github.com/pi-unnerup/jenkins.git', branch: "master"
            }
        }
        stage('Run integration tests') {
            steps {
                sh "echo hello"
            }
        }
    }
}