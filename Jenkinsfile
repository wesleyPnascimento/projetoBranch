pipeline {
    agent any

    environment {
        REPO_NAME    = "projetoBranch"
        REMOTE_USER  = "root"
        REMOTE_DIR   = "/var/www/projetoBranch"
        SSH_KEY_ACCESS = "~/.ssh/id_rsa"
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
        stage('Deploy') {
            steps {
                script {
                    def sshOpts = "-i ${env.SSH_KEY_ACCESS} -o StrictHostKeyChecking=no"
                    def target  = "${env.REMOTE_USER}@${env.REMOTE_HOST}"

                    // Garante que o diretório existe no servidor remoto
                    sh "ssh ${sshOpts} ${target} 'mkdir -p ${env.REMOTE_DIR}'"

                    // Copia index.html e info.json
                    sh "scp ${sshOpts} index.html info.json ${target}:${env.REMOTE_DIR}/"

                    echo ">>> Deploy concluído em ${env.REMOTE_HOST}:${env.REMOTE_DIR}"
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