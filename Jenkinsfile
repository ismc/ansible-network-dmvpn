pipeline {
    agent any
    options {
      disableConcurrentBuilds()
      ansiColor('xterm')
    }
    environment {
      ANSIBLE_ROLES_PATH = "${env.WORKSPACE}"
      ANSIBLE_INVENTORY_DIR = "${env.WORKSPACE}/inventory"
    }
    stages {
        stage('Prepare Workspace') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'inventory']],
                    submoduleCfg: [],
                    userRemoteConfigs: [[credentialsId: 'scarter-jenkins_key', url: 'git@github.com:ismc/inventory-scarter.git']]])
                sh 'ln -sf $PWD network-dmvpn'
            }
        }
        stage('Deploy DMVPN') {
            steps {
                echo 'Configure DMVPN...'
                  ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-test.yml", playbook: 'network-dmvpn/tests/network-dmvpn.yml'
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Check DMVPN...'
                  ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-test.yml", playbook: 'network-dmvpn/tests/network-dmvpn-check.yml'
            }
        }
    }
    post {
        always {
            echo 'Clean Workspace'
        }
    }
}
