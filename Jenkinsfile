pipeline {
    parameters {
        booleanParam defaultValue: true,
        description: 'Whether to upload the autopostgresqlbackup in playground repositories',
        name: 'PLAYGROUND'
    }
    options {
        skipDefaultCheckout()
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timeout(time: 1, unit: 'HOURS')
    }
    agent {
        node {
            label 'base-agent-v1'
        }
    }
    environment {
        NETWORK_OPTS = '--network ci_agent'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                }
                sh 'mkdir staging'
                sh 'cp -r autopostgresqlbackup staging'
                stash includes: 'staging/**', name: 'staging'
            }
        }
        stage('Build deb/rpm') {
            stages {
                stage('yap') {
                    parallel {
                        stage('Ubuntu 20.04') {
                            agent {
                                node {
                                    label 'yap-agent-ubuntu-20.04-v2'
                                }
                            }
                            steps {
                                unstash 'staging'
                                sh 'cp -r staging /tmp'
                                script {
                                    if (BRANCH_NAME == 'devel') {
                                        def timestamp = new Date().format('yyyyMMddHHmmss')
                                        sh "sudo yap build ubuntu-focal /tmp/staging/autopostgresqlbackup -r ${timestamp}"
                                    } else {
                                        sh 'sudo yap build ubuntu-focal /tmp/staging/autopostgresqlbackup'
                                    }
                                }
                                stash includes: 'artifacts/', name: 'artifacts-ubuntu-focal'
                            }
                            post {
                                always {
                                    archiveArtifacts artifacts: 'artifacts/*focal*.deb', fingerprint: true
                                }
                            }
                        }
                        stage('Ubuntu 22.04') {
                            agent {
                                node {
                                    label 'yap-agent-ubuntu-22.04-v2'
                                }
                            }
                            steps {
                                unstash 'staging'
                                sh 'cp -r staging /tmp'
                                script {
                                    if (BRANCH_NAME == 'devel') {
                                        def timestamp = new Date().format('yyyyMMddHHmmss')
                                        sh "sudo yap build ubuntu-jammy /tmp/staging/autopostgresqlbackup -r ${timestamp}"
                                    } else {
                                        sh 'sudo yap build ubuntu-jammy /tmp/staging/autopostgresqlbackup'
                                    }
                                }
                                stash includes: 'artifacts/', name: 'artifacts-ubuntu-jammy'
                            }
                            post {
                                always {
                                    archiveArtifacts artifacts: 'artifacts/*jammy*.deb', fingerprint: true
                                }
                            }
                        }
                        stage('Ubuntu 24.04') {
                            agent {
                                node {
                                    label 'yap-agent-ubuntu-24.04-v2'
                                }
                            }
                            steps {
                                unstash 'staging'
                                sh 'cp -r staging /tmp'
                                script {
                                    if (BRANCH_NAME == 'devel') {
                                        def timestamp = new Date().format('yyyyMMddHHmmss')
                                        sh "sudo yap build ubuntu-noble /tmp/staging/autopostgresqlbackup -r ${timestamp}"
                                    } else {
                                        sh 'sudo yap build ubuntu-noble /tmp/staging/autopostgresqlbackup'
                                    }
                                }
                                stash includes: 'artifacts/', name: 'artifacts-ubuntu-noble'
                            }
                            post {
                                always {
                                    archiveArtifacts artifacts: 'artifacts/*noble*.deb', fingerprint: true
                                }
                            }
                        }
                        stage('Rocky 8') {
                            agent {
                                node {
                                    label 'yap-agent-rocky-8-v2'
                                }
                            }
                            steps {
                                unstash 'staging'
                                sh 'cp -r staging /tmp'
                                script {
                                    if (BRANCH_NAME == 'devel') {
                                        def timestamp = new Date().format('yyyyMMddHHmmss')
                                        sh "sudo yap build rocky-8 /tmp/staging/autopostgresqlbackup -r ${timestamp}"
                                    } else {
                                        sh 'sudo yap build rocky-8 /tmp/staging/autopostgresqlbackup'
                                    }
                                }
                                stash includes: 'artifacts/x86_64/*el8*.rpm', name: 'artifacts-rocky-8'
                            }
                            post {
                                always {
                                    archiveArtifacts artifacts: 'artifacts/x86_64/*el8*.rpm', fingerprint: true
                                }
                            }
                        }
                        stage('Rocky 9') {
                            agent {
                                node {
                                    label 'yap-agent-rocky-9-v2'
                                }
                            }
                            steps {
                                unstash 'staging'
                                sh 'cp -r staging /tmp'
                                script {
                                    if (BRANCH_NAME == 'devel') {
                                        def timestamp = new Date().format('yyyyMMddHHmmss')
                                        sh "sudo yap build rocky-9 /tmp/staging/autopostgresqlbackup -r ${timestamp}"
                                    } else {
                                        sh 'sudo yap build rocky-9 /tmp/staging/autopostgresqlbackup'
                                    }
                                }
                                stash includes: 'artifacts/x86_64/*el9*.rpm', name: 'artifacts-rocky-9'
                            }
                            post {
                                always {
                                    archiveArtifacts artifacts: 'artifacts/x86_64/*el9*.rpm', fingerprint: true
                                }
                            }
                        }
                    }
                }
            }
        }
        stage('Upload To Playground') {
            when {
                expression { params.PLAYGROUND == true }
            }
            steps {
                unstash 'artifacts-ubuntu-focal'
                unstash 'artifacts-ubuntu-jammy'
                unstash 'artifacts-ubuntu-noble'
                unstash 'artifacts-rocky-8'
                unstash 'artifacts-rocky-9'

                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec
                    buildInfo = Artifactory.newBuildInfo()
                    uploadSpec = """{
                        "files": [
                            {
                                "pattern": "artifacts/*focal*.deb",
                                "target": "ubuntu-playground/pool/",
                                "props": "deb.distribution=focal;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/*jammy*.deb",
                                "target": "ubuntu-playground/pool/",
                                "props": "deb.distribution=jammy;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/*noble*.deb",
                                "target": "ubuntu-playground/pool/",
                                "props": "deb.distribution=noble;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/x86_64/(carbonio-autopostgresqlbackup)-(*).el8.x86_64.rpm",
                                "target": "centos8-playground/zextras/{1}/{1}-{2}.el8.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}",
                                "exclusions": ["*alertmanager*.rpm","*exporter*.rpm"]
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                }
            }
        }
        stage('Upload To Devel') {
            when {
                branch 'devel';
            }
            steps {
                unstash 'artifacts-ubuntu-focal'
                unstash 'artifacts-ubuntu-jammy'
                unstash 'artifacts-ubuntu-noble'
                unstash 'artifacts-rocky-8'
                unstash 'artifacts-rocky-9'

                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec
                    buildInfo = Artifactory.newBuildInfo()
                    uploadSpec = """{
                        "files": [
                            {
                                "pattern": "artifacts/*focal*.deb",
                                "target": "ubuntu-devel/pool/",
                                "props": "deb.distribution=focal;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/*jammy*.deb",
                                "target": "ubuntu-devel/pool/",
                                "props": "deb.distribution=jammy;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/*noble*.deb",
                                "target": "ubuntu-devel/pool/",
                                "props": "deb.distribution=noble;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/x86_64/(carbonio-autopostgresqlbackup)-(*).el8.x86_64.rpm",
                                "target": "centos8-devel/zextras/{1}/{1}-{2}.el8.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}",
                                "exclusions": ["*alertmanager*.rpm","*exporter*.rpm"]
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                }
            }
        }
        stage('Upload & Promotion Config') {
            when {
                anyOf {
                    branch 'release/*'
                    buildingTag()
                }
            }
            steps {
                unstash 'artifacts-ubuntu-focal'
                unstash 'artifacts-ubuntu-jammy'
                unstash 'artifacts-ubuntu-noble'
                unstash 'artifacts-rocky-8'
                unstash 'artifacts-rocky-9'

                script {
                    def server = Artifactory.server 'zextras-artifactory'
                    def buildInfo
                    def uploadSpec
                    def config

                    //ubuntu
                    buildInfo = Artifactory.newBuildInfo()
                    buildInfo.name += "-ubuntu"
                    uploadSpec = """{
                        "files": [
                            {
                                "pattern": "artifacts/*focal*.deb",
                                "target": "ubuntu-rc/pool/",
                                "props": "deb.distribution=focal;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/*jammy*.deb",
                                "target": "ubuntu-rc/pool/",
                                "props": "deb.distribution=jammy;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            },
                            {
                                "pattern": "artifacts/*noble*.deb",
                                "target": "ubuntu-rc/pool/",
                                "props": "deb.distribution=noble;deb.component=main;deb.architecture=amd64;vcs.revision=${env.GIT_COMMIT}"
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                    config = [
                            'buildName'          : buildInfo.name,
                            'buildNumber'        : buildInfo.number,
                            'sourceRepo'         : 'ubuntu-rc',
                            'targetRepo'         : 'ubuntu-release',
                            'comment'            : 'Do not change anything! Just press the button',
                            'status'             : 'Released',
                            'includeDependencies': false,
                            'copy'               : true,
                            'failFast'           : true
                    ]
                    Artifactory.addInteractivePromotion server: server, promotionConfig: config, displayName: "Ubuntu Promotion to Release"
                    server.publishBuildInfo buildInfo


                    //rocky8
                    buildInfo = Artifactory.newBuildInfo()
                    buildInfo.name += "-centos8"
                    uploadSpec= """{
                        "files": [
                            {
                                "pattern": "artifacts/x86_64/(carbonio-autopostgresqlbackup)-(*).el8.x86_64.rpm",
                                "target": "centos8-rc/zextras/{1}/{1}-{2}.el8.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}",
                                "exclusions": ["*alertmanager*.rpm","*exporter*.rpm"]
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                    config = [
                            'buildName'          : buildInfo.name,
                            'buildNumber'        : buildInfo.number,
                            'sourceRepo'         : 'centos8-rc',
                            'targetRepo'         : 'centos8-release',
                            'comment'            : 'Do not change anything! Just press the button',
                            'status'             : 'Released',
                            'includeDependencies': false,
                            'copy'               : true,
                            'failFast'           : true
                    ]
                    Artifactory.addInteractivePromotion server: server, promotionConfig: config, displayName: "Centos8 Promotion to Release"
                    server.publishBuildInfo buildInfo

                    //rocky9
                    buildInfo = Artifactory.newBuildInfo()
                    buildInfo.name += "-rhel9"
                    uploadSpec= """{
                        "files": [
                            {
                                "pattern": "artifacts/x86_64/(carbonio-autopostgresqlbackup)-(*).el9.x86_64.rpm",
                                "target": "rhel9-rc/zextras/{1}/{1}-{2}.el9.x86_64.rpm",
                                "props": "rpm.metadata.arch=x86_64;rpm.metadata.vendor=zextras;vcs.revision=${env.GIT_COMMIT}",
                                "exclusions": ["*alertmanager*.rpm","*exporter*.rpm"]
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec, buildInfo: buildInfo, failNoOp: false
                    config = [
                            'buildName'          : buildInfo.name,
                            'buildNumber'        : buildInfo.number,
                            'sourceRepo'         : 'rhel9-rc',
                            'targetRepo'         : 'rhel9-release',
                            'comment'            : 'Do not change anything! Just press the button',
                            'status'             : 'Released',
                            'includeDependencies': false,
                            'copy'               : true,
                            'failFast'           : true
                    ]
                    Artifactory.addInteractivePromotion server: server, promotionConfig: config, displayName: "Rhel9 Promotion to Release"
                    server.publishBuildInfo buildInfo
                }
            }
        }
    }
}