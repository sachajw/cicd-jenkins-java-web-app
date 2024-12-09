pipeline {
    agent {
        label 'jenkins-jenkins-agent'
    }
    environment {
        DOCKERREPO = 'quay.io/pangarabbit/ortelius-jenkins-demo-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.substring(0, 7)}"
        DISCORD = credentials('pangarabbit-discord-jenkins')
        JDK17_CONTAINER = 'agent-jdk17'
        PYTHON_CONTAINER = 'python39'
        KANIKO_CONTAINER = 'kaniko'
        DHUSER = credentials('dh-pangarabbit')
        DHPASS = credentials('dh-pangarabbit')
        DHORG = "PangaRabbit"
        DHPROJECT = "ortelius-jenkins-demo-app"
        DHURL = "https://ortelius.pangarabbit.com"
    }

    stages {
        stage('Git Checkout') {
            steps {
                container("${JDK17_CONTAINER}") {
                    withCredentials([string(credentialsId: 'gh-sachajw-walle-secret-text', variable: 'GITHUB_PAT')]) {
                        sh "git config --global --add safe.directory ${env.WORKSPACE} && \
                        git clone https://'${GITHUB_PAT}'@github.com/sachajw/ortelius-jenkins-demo-app.git"
                    }
                }
            }
        }
    }

        // stage('Surefire Report') {
        //     steps {
        //         container("${JDK17_CONTAINER}") {
        //             sh '''
        //                 ./mvnw clean install site surefire-report:report -Dcheckstyle.skip=true
        //             '''
        //         }
        //     }
        // }

    //    stage('Ortelius') {
    //         steps {
    //             container("${PYTHON_CONTAINER}") {
    //                 sh '''
    //                     #curl -sSL https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    //                     #python get-pip.py
    //                     pip install ortelius-cli
    //                     dh envscript --envvars component.toml --envvars_sh ${WORKSPACE}/dhenv.sh

    //                     echo Logging into Docker
    //                     echo ${DHPASS} | docker login -u ${DHUSER} --password-stdin ${DHURL}

    //                     echo Building and Pushing Docker Image
    //                     . ${WORKSPACE}/dhenv.sh
    //                     docker build --tag ${DOCKERREPO}:${IMAGE_TAG} .
    //                     docker push ${DOCKERREPO}:${IMAGE_TAG}
    //                     echo export DIGEST=$(docker inspect --format='{{index .RepoDigests 0}}' ${DOCKERREPO}:${IMAGE_TAG} | cut -d: -f2 | cut -c-12) >> ${WORKSPACE}/dhenv.sh

    //                     echo Capturing SBOM
    //                     . ${WORKSPACE}/dhenv.sh
    //                     curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b .
    //                     ./syft packages ${DOCKERREPO}:${IMAGE_TAG} --scope all-layers -o cyclonedx-json > ${WORKSPACE}/cyclonedx.json
    //                     cat ${WORKSPACE}/cyclonedx.json

    //                     echo Creating Component with Build Data and SBOM
    //                     . ${WORKSPACE}/dhenv.sh
    //                     dh updatecomp --rsp component.toml --deppkg "cyclonedx@${WORKSPACE}/cyclonedx.json"
    //                 '''
    //             }
    //         }
    //     }
    // }

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
