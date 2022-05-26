pipeline {
    agent any
    environment {
        BRANCH = 'PS-15'
        CF_HOST = 'https://g.codefresh.io'
        CODEFRESH_VERSION = '0.0.6'
        DOCKER_PASSWORD = credentials('docker-hub-registry-password')
        DOCKER_USERNAME = 'dustinvanbuskirk'
        IMAGE_NAME = 'dustinvanbuskirk/jenkins-example-app'
        IMAGE = 'docker.io/dustinvanbuskirk/jenkins-example-app:0.0.2'
        IMAGE_SHA = 'docker.io/dustinvanbuskirk/jenkins-example-app:0.0.2'
        IMAGE_URI = 'docker.io/dustinvanbuskirk/jenkins-example-app:0.0.2'
        GIT_BRANCH = 'main'
        GIT_COMMIT_MESSAGE = 'Jenkins pushed this commit'
        GIT_COMMIT_URL = 'https://github.com/codefresh-contrib/jenkins-codefresh-gitops/commit/3698de5abf39380ed35ed33b59435e904e3eff42'
        GIT_REVISION = '3698de5abf39380ed35ed33b59435e904e3eff42'
        GIT_SENDER_LOGIN='dustin@codefresh.io'
        GITHUB_TOKEN=credentials('github-personal-access-token')
        GITHUB_SHA='3698de5abf39380ed35ed33b59435e904e3eff42'
        JIRA_PROJECT_PREFIX = 'PS'
        JIRA_HOST = 'codefresh-io.atlassian.net'
        JIRA_EMAIL = 'dustin@codefresh.io'
        JIRA_API_TOKEN = credentials('jira-token')
        LOGS_URL = "${env.BUILD_URL}"
        MESSAGE = 'PS-15'
        REPO  = 'codefresh-contrib/jenkins-codefresh-gitops'
        WORKFLOW_NAME = "${env.JOB_NAME}"
        WORKFLOW_URL = "${env.JOB_URL}"
    }
    stages {
        stage('Cloning Git') {
          steps {
            git([url: 'https://github.com/codefresh-contrib/jenkins-codefresh-gitops-example.git', branch: 'main', credentialsId: 'github-salesdemo'])

          }
        }
        stage('Building image') {
          steps{
            script {
              dockerImage = docker.build IMAGE_NAME
            }
          }
        }
        stage('Deploy Image') {
          steps{
            script {
              docker.withRegistry( '', dockerhub-creds ) {
                dockerImage.push("0.0.$BUILD_NUMBER")
              }
            }
          }
        }
        stage('Report Image to Codefresh') {
            agent{
                docker {
                    image "quay.io/codefreshplugins/argo-hub-workflows-codefresh-csdp-versions-${env.CODEFRESH_VERSION}-images-report-image-info:main"
                    args "--net='host'"
                }
            }
                environment { 
                    CF_API_KEY = credentials('codefresh-runtime-token')
                }
                steps {
                      sh 'node /usr/src/app/index.js'
                }
        }
        stage('Enrich GitHub Pull Request') {
            agent{
                docker {
                    image "quay.io/codefreshplugins/argo-hub-workflows-codefresh-csdp-versions-${env.CODEFRESH_VERSION}-images-image-enricher-git-info:main"
                    args "--net='host'"
                }
            }
            environment { 
                CF_API_KEY = credentials('codefresh-platform-token')
            }
            steps {
                  sh 'node /app/src/index.js'
            }
        }
        stage('Enrich JIRA') {
            agent{
                docker {
                    image "quay.io/codefreshplugins/argo-hub-workflows-codefresh-csdp-versions-${env.CODEFRESH_VERSION}-images-image-enricher-jira-info:main"
                    args "--net='host'"
                }
            }
            environment { 
                CF_API_KEY = credentials('codefresh-platform-token')
            }
            steps {
                  sh 'node /app/src/index.js'
            }
        }
    }
}
