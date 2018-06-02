pipeline {
    agent any
    options {
      disableConcurrentBuilds()
      skipDefaultCheckout()
      ansiColor('xterm')
      lock resource: 'shared_resource_lock'
    }
    environment {
      ANSIBLE_ROLES_PATH = "${env.WORKSPACE}"
      ANSIBLE_INVENTORY_DIR = "${env.WORKSPACE}/inventory"
    }
    stages {
        stage('Prepare Workspace') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: scm.branches,
                    doGenerateSubmoduleConfigurations: false,
                    extensions: scm.extensions + [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'network-dmvpn']],
                    userRemoteConfigs: scm.userRemoteConfigs
                ])
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'inventory']],
                    submoduleCfg: [],
                    userRemoteConfigs: [[credentialsId: 'scarter-jenkins_key', url: 'git@github.com:ismc/inventory-test.git']]
                ])
            }
        }
        stage('Deploy DMVPN') {
            steps {
                echo 'Configure DMVPN...'
                  ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/jenkins-wan-testbed.yml", playbook: 'network-dmvpn/tests/network-dmvpn.yml'
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Check DMVPN...'
                  ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/jenkins-wan-testbed.yml", playbook: 'network-dmvpn/tests/network-dmvpn-check.yml'
            }
        }
    }
    post {
        always {
            echo 'Clean Workspace'
        }
    }
}
