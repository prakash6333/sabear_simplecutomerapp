pipeline {
    agent any
    tools {
        maven 'MVN_HOME'   // must match Maven tool name in Jenkins
    }
    environment {
        SCANNER_HOME = tool 'SonarScanner'   // must match SonarScanner tool name
    }
    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'master', url: 'https://github.com/prakash6333/sabear_simplecutomerapp.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {   // must match SonarQube server name in Jenkins
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=${BUILD_NUMBER} \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target
                    """
                }
            }
        }
        stage('Publish to Nexus') {
            steps {
                sh 'mvn deploy'
            }
        }
       stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'admin', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T target/SimpleCustomerApp.war \
                         http://107.21.137.31:8080/manager/text/deploy?path=/hiring&update=true
                    '''
                }
            }
        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-integration',
                    color: 'good',
                    message: "✅ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} finished successfully",
                    tokenCredentialId: 'slack_integration'
                )
            }
        }
    }
    post {
        failure {
            slackSend(
                channel: '#jenkins-integration',
                color: 'danger',
                message: "❌ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed!",
                tokenCredentialId: 'slack_integration'
            )
        }
    }
}
}
