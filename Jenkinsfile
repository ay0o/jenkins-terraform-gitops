#!/usr/bin/env groovy

pipeline {

    agent {
        label 'deploy'
    }

    triggers {
        gitlab(
            triggerOnPush: true,
            triggerOnMergeRequest: true,
            triggerOnNoteRequest: true,
            triggerOpenMergeRequestOnPush: "never",
            noteRegex: "/rebuild",
            skipWorkInProgressMergeRequest: true,
            ciSkip: true,
            setBuildDescription: true,
            addNoteOnMergeRequest: false,
            addCiMessage: false,
            addVoteOnMergeRequest: false,
            acceptMergeRequestOnSuccess: false,
            branchFilterType: "NameBasedFilter",
            includeBranchesSpec: "master",
            excludeBranchesSpec: "",
            secretToken: null
        )
    }

    options {
        ansiColor('xterm')
        buildDiscarder(logRotator(numToKeepStr: '100'))
        disableConcurrentBuilds()
        skipDefaultCheckout()
        timeout(time: 1, unit: 'HOURS')
        gitLabConnection('GITLAB')
        timestamps()
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    if (env.gitlabMergeRequestId) {
                        git branch: env.gitlabSourceBranch, credentialsId: <GITLAB_CREDENTIALS>, url: <REPO>
                    }
                    else {
                        git credentialsId: <GITLAB_CREDENTIALS>, url: <REPO>
                    }
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Code Tests') {
            when {
                expression { env.gitlabMergeRequestId }
            }
            steps {
                script {
                    def fmtStatus = sh (script: 'terraform fmt -check', returnStatus: true)
                    def validateStatus = sh (script: 'terraform validate', returnStatus: true)
                    def status = (fmtStatus == 0 && validateStatus == 0)
                    if (!status) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            when {
                expression { ! env.gitlabMergeRequestId }
            }
            steps {
                sh 'terraform apply -auto-approve'
            }
        }

    }
    post {
        cleanup {
            cleanWs()
        }
    }
}
