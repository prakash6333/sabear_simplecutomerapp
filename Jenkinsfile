pipeline {
    agent any

    tools {
        maven "MVN_HOME"   // Must match the Maven tool configured in Jenkins
    }

    environment {
        NEXUS_VERSION       = "nexus3"
        NEXUS_PROTOCOL      = "http"
        NEXUS_URL           = "52.91.43.63:8081"   // Nexus IP
        NEXUS_REPOSITORY    = "simple-app"
        NEXUS_CREDENTIAL_ID = "nexus"             // Nexus Jenkins credential ID
        TOMCAT_CRED         = "tomcat_credentials" // Tomcat Jenkins credential ID
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'feature-1.1',
                    url: 'https://github.com/prakash6333/sabear_simplecutomerapp.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarcube_jenkins integration') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml"
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    artifactPath = filesByGlob[0].path

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: artifactPath,
                             type: pom.packaging],
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: "pom.xml",
                             type: "pom"]
                        ]
                    )
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
                             "http://107.21.137.31:8080/manager/text/deploy?path=/hiring&update=true"
                    '''
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-integration',
                    color: 'good',
                    message: "✅ Deployment successful for *Simple Customer App* on Tomcat (107.21.137.31).\nDeployed by: prakash"
                )
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        failure {
            slackSend(
                channel: '#jenkins-integration',
                color: 'danger',
                message: "⚠️ Jenkins pipeline for *Simple Customer App* failed! Please check logs."
            )
        }
    }
}
