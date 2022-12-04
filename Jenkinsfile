pipeline {
    options {
        disableConcurrentBuilds();
        ansiColor('xterm');
        buildDiscarder(logRotator(numToKeepStr: '30'));
    }

    agent {
        label 'generic'
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }
        stage('Build WAR file') {
            steps {
                sh "sbt executable"
                archiveArtifacts artifacts: 'target/executable/*.war', fingerprint: true
            }
        }
        stage('Deploy') {
            when {
                branch "master"
            }
            steps {
                sh "echo 'Deployment...'"
            }
        }
    }

    post {
        cleanup {
            cleanWs()
        }
    }
}