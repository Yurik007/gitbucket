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
        stage('Run tests') {
            steps {
                sh """sbt coverage test
                sbt coverageReport"""
            }
        }
        stage('Execute SonarQube analysis') {
            environment {
                scannerHome = tool 'sonar-scanner';
            }
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh """#!/bin/bash -l
                        ${scannerHome}/bin/sonar-scanner -Dsonar.sourceEncoding=UTF-8 \\
                        -Dsonar.projectKey=gitbucket \\
                        -Dsonar.projectName=gitbucket \\
                        -Dsonar.exclusions=**/*.java \\
                        -Dsonar.scala.coverage.reportPaths=target/scala-2.13/scoverage-report/scoverage.xml"""
                    }

                    timeout(time: 5, unit: 'MINUTES') {
                        sleep 15
                        def qg = waitForQualityGate()
                        currentBuild.description = "Quality Gate ${qg.status}"
                        if (qg.status != "OK") {
                            if (env.BRANCH_NAME != 'master') {
                                unstable "Quality Gate ${qg.status}. Please, address new issues"
                            } else {
                                failure "Quality Gate ${qg.status}. Please, address new issues"
                            }
                        }
                    }
                }
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