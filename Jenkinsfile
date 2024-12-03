pipeline {
    agent {
        label 'jenkins-jenkins-agent'
    }
    environment {
        DOCKERREPO = 'quay.io/pangarabbit/ortelius-jenkins-demo-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.substring(0, 7)}"
        DISCORD = credentials('pangarabbit-discord-jenkins')
        DEFAULT_CONTAINER = 'agent-jdk17'
        KANIKO_CONTAINER = 'kaniko'
    }

    stages {
        stage('Git Checkout') {
            steps {
                container("${DEFAULT_CONTAINER}") {
                    withCredentials([string(credentialsId: 'gh-sachajw-walle-secret-text', variable: 'GITHUB_PAT')]) {
                        sh "git config --global --add safe.directory ${env.WORKSPACE} && \
                        git clone https://'${GITHUB_PAT}'@github.com/sachajw/ortelius-jenkins-demo-app.git"
                    }
                }
            }
        }

        stage('Surefire Report') {
            steps {
                container("${DEFAULT_CONTAINER}") {
                    sh '''
                        ./mvnw clean install site surefire-report:report -Dcheckstyle.skip=true
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Publishing HTML Report'
            publishHTML(target: [
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/site',
            reportFiles: 'surefire-report.html',
            reportName: 'Surefire Reports'
            ])
        }

        always {
            echo 'Sending Discord Notification'
            withCredentials([string(credentialsId: 'pangarabbit-discord-jenkins', variable: 'DISCORD')]) {
                discordSend description: """
                            Result: ${currentBuild.currentResult}
                            Service: ${env.JOB_NAME}
                            Build Number: [#${env.BUILD_NUMBER}](${env.BUILD_URL})
                            Branch: ${env.GIT_BRANCH}
                            Commit User: ${env.GIT_COMMIT_USER}
                            Duration: ${currentBuild.durationString}
                        """,
                        footer: 'Wall-E loves you!',
                        link: env.BUILD_URL,
                        result: currentBuild.currentResult,
                        title: env.JOB_NAME,
                        webhookURL: DISCORD
                }
            }
        }
    }
