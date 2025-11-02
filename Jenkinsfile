pipeline {
    agent any
    stages {
        stage('Clean WorkSpace') {
            steps {
                // Clean before build
                cleanWs()
                // We need to explicitly checkout from SCM here
                checkout scm
                echo "Building ${env.JOB_NAME}..."
            }
        }

        stage('Frontend: Build') {
            steps {
                echo "Installing frontend dependencies and building..."
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Frontend: Docker Build') {
            steps {
                script {
                    echo "Building Docker image for frontend..."
                    def packageJson = readJSON file: 'package.json'
                    withCredentials([usernamePassword(credentialsId: 'hub.docker.com', passwordVariable: 'HUB_REPO_PASS', usernameVariable: 'HUB_REPO_USER')]) {
                        def user = env.HUB_REPO_USER
                        def password = env.HUB_REPO_PASS
                        sh "docker version"
                        sh "docker login -u $user -p $password"

                        // Build image with both tags
                        sh "docker build -t maliciamrg/${packageJson.name.toLowerCase()}:${packageJson.version} ."

                        // Always push versioned tag
                        sh "docker push maliciamrg/${packageJson.name.toLowerCase()}:${packageJson.version}"

                        // Push 'latest' only if master branch
                        if (env.BRANCH_NAME == 'master') {
                            sh "docker tag maliciamrg/${packageJson.name.toLowerCase()}:${packageJson.version} maliciamrg/${packageJson.name.toLowerCase()}:latest"
                            sh "docker push maliciamrg/${packageJson.name.toLowerCase()}:latest"
                        }

                        sleep 10 // Wait for 10 seconds
                    }
                }
            }
        }

        stage("assistantPhoto : Deploy to Docker Server") {
            steps {
                script {
                    sh "docker compose -p frontend down"
                    sh "docker compose -p frontend up -d --force-recreate"
                }
            }
        }

    }
    post {
        always {
            print "always"
        }
        changed {
            print "changed"
        }
        fixed {
            print "fixed"
            discordSend(description: "Jenkins Pipeline Build",
                    footer: "Status fixed",
                    link: env.BUILD_URL,
                    result: currentBuild.currentResult,
                    title: JOB_NAME,
                    webhookURL: "https://discord.com/api/webhooks/1251803129004032030/Ms-4v3aw3MMkIHIECMYMiP48NTV_F1IazsvwQmAqGGFw4OOR9FRX-DwjFG5V1dV-zKg6")
        }
        regression {
            print "regression"
        }
        aborted {
            print "aborted"
        }
        failure {
            print "failure"
            script {
                if (!currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')) {
                    discordSend(description: "Jenkins Pipeline Build",
                            footer: "Status failure",
                            link: env.BUILD_URL,
                            result: currentBuild.currentResult,
                            title: JOB_NAME,
                            webhookURL: "https://discord.com/api/webhooks/1251803129004032030/Ms-4v3aw3MMkIHIECMYMiP48NTV_F1IazsvwQmAqGGFw4OOR9FRX-DwjFG5V1dV-zKg6")
                }
            }
        }
        success {
            print "success"
        }
        unstable {
            print "unstable"
        }
        unsuccessful {
            print "unsuccessful"
        }
        cleanup {
            print "cleanup"
        }
    }
}