def createDeployRemote() {
    def remote = [: ]
    remote.name = "${env.REMOTE_NAME}"
    remote.host = "${env.REMOTE_HOST}"
    remote.user = "${env.REMOTE_USER}"
    remote.identityFile = "/var/lib/jenkins/.ssh/id_rsa"
    remote.allowAnyHosts = true
    remote.timeoutSec = 10
    remote.retryCount = 3
    remote.retryWaitSec = 3
    return remote
}

pipeline {
    agent any

    environment {
        REPO_NAME    = "projetoBranch"
        REMOTE_USER  = "root"
        REMOTE_DIR   = "/var/www/projetoBranch"
    }

    stages {

        // ── 1. Definir servidor alvo conforme a branch ──────────────────────
        stage('Set Target Server') {
            steps {
                script {
                    switch (env.BRANCH_NAME) {
                        case 'main':
                            env.REMOTE_HOST = '192.168.0.30'
                            break
                        case 'develop':
                            env.REMOTE_HOST = '192.168.0.30'
                            break
                        case 'staging':
                            env.REMOTE_HOST = '192.168.0.30'
                            break
                        case 'demo':
                            env.REMOTE_HOST = '192.168.0.30'
                            break
                        default:
                            error("Branch '${env.BRANCH_NAME}' não mapeada para nenhum servidor. Pipeline abortado.")
                    }
                    echo ">>> Branch: ${env.BRANCH_NAME} | Servidor: ${env.REMOTE_HOST}"
                }
            }
        }

        // ── 2. Checkout do repositório ──────────────────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ── 3. Gerar info.json com dados do build ───────────────────────────
        stage('Generate info.json') {
            steps {
                script {
                    def commitHash = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()

                    def infoJson = """{
    "repo": "${env.REPO_NAME}",
    "branch": "${env.BRANCH_NAME}",
    "commit": "${commitHash}"
}"""
                    writeFile file: 'info.json', text: infoJson
                    echo ">>> info.json gerado:"
                    sh "cat info.json"
                }
            }
        }

        // ── 4. Deploy: enviar arquivos para o servidor remoto ───────────────
            // stage('Deploy') {
            //     steps {
            //         script {
            //             def sshOpts = "-i ${env.SSH_KEY_ACCESS} -o StrictHostKeyChecking=no"
            //             def target  = "${env.REMOTE_USER}@${env.REMOTE_HOST}"

            //             // Garante que o diretório existe no servidor remoto
            //             // sh "ssh ${sshOpts} ${target} 'mkdir -p ${env.REMOTE_DIR}'"

            //             withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key-prod', keyFileVariable: 'SSH_KEY')]) {
            //                 sh """
            //                     ssh -i $SSH_KEY -o StrictHostKeyChecking=no root@${env.REMOTE_HOST} 'mkdir -p /var/www/projetoBranch'
            //                 """
            //             }

            //             // Copia index.html e info.json
            //             sh "scp ${sshOpts} index.html info.json ${target}:${env.REMOTE_DIR}/"

            //             echo ">>> Deploy concluído em ${env.REMOTE_HOST}:${env.REMOTE_DIR}"
            //         }
            //     }
        // }
        stage('Deploy') {
            steps {
                script {
                    def remote = createDeployRemote()
                    sshPut remote: remote, 
                        from: 'index.html', 
                        into: "${env.REMOTE_DIR}/index.html"

                    sshPut remote: remote, 
                        from: 'info.json', 
                        into: "${env.REMOTE_DIR}/info.json"

                    echo ">>> Deploy concluído em ${remote.host}:${env.REMOTE_DIR}"
                }
            }
        }

    }

    // ── Pós-execução ────────────────────────────────────────────────────────
    post {
        success {
            echo """
╔══════════════════════════════════════╗
║  ✅  PIPELINE CONCLUÍDO COM SUCESSO  ║
╠══════════════════════════════════════╣
║  Branch  : ${env.BRANCH_NAME}
║  Servidor: ${env.REMOTE_HOST}
║  Diretório: ${env.REMOTE_DIR}
╚══════════════════════════════════════╝
"""
        }
        failure {
            echo """
╔══════════════════════════════════════╗
║  ❌  PIPELINE FALHOU                 ║
╠══════════════════════════════════════╣
║  Branch  : ${env.BRANCH_NAME}
║  Servidor: ${env.REMOTE_HOST}
╚══════════════════════════════════════╝
"""
        }
    }
}