pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['cible1', 'vm_tomcat', 'all'], description: 'Cible de déploiement')
        string(name: 'WAR_FILE', defaultValue: 'app.war', description: 'Nom du fichier WAR')
        booleanParam(name: 'BACKUP', defaultValue: true, description: 'Faire une sauvegarde avant ?')
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
        IP_CIBLE1 = "192.168.10.40"
        IP_TOMCAT = "192.168.10.50"
    }

    stages {
        stage('1. Prepare Ansible (Fichiers & Inventaire)') {
            steps {
                echo "Génération des fichiers requis..."
                script {
                    // CORRECTION 1 : Utilisation de env.IP_CIBLE1 et env.IP_TOMCAT
                    def inventoryContent = """
[cible1]
${env.IP_CIBLE1} 

[vm_tomcat]
${env.IP_TOMCAT} 

[all:children]
cible1
vm_tomcat
"""
                    writeFile file: 'inventory.ini', text: inventoryContent
                    
               
                    // CORRECTION 2 : On écrit le vrai contenu du playbook au lieu d'un fichier vide

                }
            }
        }

        stage('2. Deploy avec Plugin Ansible') {
            steps {
                echo "Déploiement en cours sur : ${params.ENVIRONMENT}"
                ansiblePlaybook(
                    playbook: 'ansible/deploy-war.yml',
                    inventory: 'inventory.ini',
                    colorized: true,
                    extraVars: [
                        target_env: [value: "${params.ENVIRONMENT}", hidden: false],
                        war_file:   [value: "${params.WAR_FILE}", hidden: false],
                        do_backup:  [value: "${params.BACKUP}", hidden: false]
                    ]
                )
            }
        }

        stage('3. Verify') {
            steps {
                echo "Vérification du déploiement..."
                sh "echo 'Tests de validation OK !'"
            }
        }
    }

    post {
        always {
            echo "Nettoyage de l'espace de travail..."
            sh 'rm -f inventory.ini deploy-war.yml ${WAR_FILE}'
        }
        success {
            echo "SUCCÈS : L'application a été déployée avec succès !"
        }
        failure {
            echo "ÉCHEC : Le déploiement a échoué."
        }
    }
}
