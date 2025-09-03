pipeline {
    agent any
    environment {
        NEXUS_CRED = 'nexus'
        TOMCAT_CRED = 'tomcat_credentials'
    }
    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/prakash6333/sabear_simplecutomerapp.git', branch: 'feature-1.1'
            }
        }
        stage('Build') {
            steps {
                tool name: 'Maven-3.8.4', type: 'maven'
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CRED}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                    mvn deploy -DskipTests \
                        -Dnexus.username=$NEXUS_USER \
                        -Dnexus.password=$NEXUS_PASS \
                        --settings /var/lib/jenkins/.m2/settings.xml
                    '''
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${TOMCAT_CRED}", usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                    WAR_FILE=$(ls target/*.war | head -n 1)
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T $WAR_FILE \
                         http://18.234.209.4:8080/manager/text/deploy?path=/hiring&update=true
                    '''
                }
            }
        }
        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-integration',
                    color: 'good',
                    message: "Hi Team, Jenkins pipeline for jenkins-04 task 2 *SIMPLE CUSTOMER APP* has finished successfully! :white_tick:\nDeployed by: prakash"
                )
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'
        }
        failure {
            slackSend(
                channel: '#jenkins-integration',
                color: 'danger',
                message: ":warning: Jenkins pipeline for *hiring-app* failed! Please check."
            )
        }
    }
}
