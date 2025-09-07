pipeline {
    agent any

    tools {
        maven 'MVN_HOME'      // This must match the name in Jenkins Global Tool Configuration
    }

    environment {
        SCANNER_HOME = tool 'SonarScanner'   // SonarScanner name from Jenkins tool config
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
                withSonarQubeEnv('SonarQube-Server') {   // must match SonarQube server name in Jenkins system config
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=${BUILD_NUMBER} \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
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
                sh '''
                    echo "Deploying WAR to Tomcat..."
                    cp target/*.war /opt/tomcat/webapps/
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
