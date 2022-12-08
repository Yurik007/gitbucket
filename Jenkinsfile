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
                            if (env.BRANCH != 'master') {
                                unstable "Quality Gate ${qg.status}. Please, address new issues"
                            } else {
                                error "Quality Gate ${qg.status}. Please, address new issues"
                            }
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH == 'master') {
                        withCredentials([usernamePassword(credentialsId: 'AWS Beanstalk CLI', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                                sh """export AWS_DEFAULT_REGION=eu-central-1
                                aws s3 cp target/executable/gitbucket.war s3://elasticbeanstalk-eu-central-1-618727779669/GitBucket/gitbucket-${BUILD_NUMBER}.war
                                aws elasticbeanstalk create-application-version --application-name GitBucket --version-label gitbucket-${BUILD_NUMBER} --source-bundle S3Bucket=elasticbeanstalk-eu-central-1-618727779669,S3Key=GitBucket/gitbucket-${BUILD_NUMBER}.war
                                aws elasticbeanstalk update-environment --application-name GitBucket --environment-name Gitbucket-env --version-label gitbucket-${BUILD_NUMBER}
                                """
                        }
                    }
                }
            }
        }
    }

    post {
        cleanup {
            cleanWs()
        }
    }
}