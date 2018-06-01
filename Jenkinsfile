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
                    userRemoteConfigs: [[credentialsId: '56de1652-01f3-4a6f-9615-e7d5aab840aa', url: 'git@github.com:ismc/inventory-scarter.git']]])
                sh 'ln -s $PWD .'
            }
        }
        stage('Run Tests') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: '56de1652-01f3-4a6f-9615-e7d5aab840aa', keyFileVariable: '', passphraseVariable: '', usernameVariable: '')]) {
                    echo 'Configure DMVPN...'
                      ansiblePlaybook colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-test.yml", playbook: 'network-dmvpn/tests/network-dmvpn.yml'
                    echo 'Check DMVPN...'
                      ansiblePlaybook colorized: true, limit: 'network', disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-test.yml", playbook: 'network-dmvpn/tests/network-dmvpn-check.yml'
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning Workspace...'
            /* deleteDir() */
        }
    }
}
