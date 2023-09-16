pipeline {
    agent {
        node {
            label 'built-in'
        }
    }
    stages {
        stage('Continuous download') {
            steps {
                //git branch: 'main', credentialsId: 'classicstan', url: 'https://github.com/classicstan/School-webapp.git'
                git branch: 'main', url: 'https://github.com/classicstan/School-webapp.git'
            }
        }
        stage ('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonarqube';
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.login=sqa_58d76e80238e92fed3510ee9b1927054cee59225 \
                            -Dsonar.projectKey=Test2 \
                            -Dsonar.exclusions=vendor/**,resources/**,**/*.java \
                            -Dsonar.sources=/var/lib/jenkins/workspace/Operations2024/src \
                            -Dsonar.host.url=http://172.31.26.120:9000"
                    }
                }
            }
        }
        stage('SonarQube QG status') {
            steps {
                sleep(time: 60, unit: 'SECONDS')
                timeout(time: 1, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error"Pipeline aborted due to quality gate failure: ${qg.status}")
                        }
                    }
                }
            }
        }
        stage('Continuous build') {
            steps {
               sh 'mvn clean package'
            }
        }
        stage('continous deployment') {
            steps {
               deploy adapters: [tomcat9(credentialsId: 'Devcredentials', path: '', url:'http://172.31.94.205:8080')], contextPath: 'qaenv', war: '**/*.war'
            }
        }
        stage('Continuous testing') {
            steps {
                echo 'testing was successful'
            }
        }
        stage('deploy artifact') {
            steps {
                nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'tt.test', 
                            classifier: '', 
                            file: '/var/lib/jenkins/workspace/Operations2024/target/tt.test.war', 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexus', 
                    groupId: 'com.tt', 
                    nexusUrl: '172.31.28.178:8081',
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'Operations2023', 
                    version: '*1.0-SNAPSHOT'
            }
        }
        stage("Download artifact from Nexus") {
            steps {
                sh 'wget --user=admin --password=Operations23 http://172.31.28.178:8081/repository/Operations2023/com/tt/tt.test/*1.0-SNAPSHOT/tt.test-*1.0-20230408.034534-7.war'
            }
        }
        stage('continous dlivery') {
            steps {
                input message: 'are you authorized to run this job?', 
                submitter: 'stanley'
                deploy adapters: [tomcat9(credentialsId: 'Prodcredentials', path: '', url: 'http://172.31.93.91:8080')], contextPath: 'prod', war: '**/*.war'
            }
        }
        stage('Post-Build Notification') {
            steps {
                script {
                    def status = currentBuild.currentResult
                    def message = "Build ${status}:Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'" 
                    slackSend (
                        channel: '#operation2023', 
                        color: status == 'SUCCESS' ? 'good' : 'danger',
                        message: message, 
                        tokenCredentialId: 'slack'
                    )
                }
            }
        }    
        
    }    
    post {
        always {
            script {
                slackSend (
                    channel: 'operation2023',
                    color: currentBuild.currentResult == 'SUCCESS' ? 'good' : 'danger',
                    message: "Build ${currentBuild.currentResult}: Job'${env.JOB_NAME} [${env.BUILD_NUMBER}]'", 
                    tokenCredentialId: 'slack'
                )
            }
        }
    }
}
