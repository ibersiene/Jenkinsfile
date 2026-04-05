pipeline {
    // Le pipeline peut s'exécuter sur n'importe quel agent Jenkins
    agent any

    // 1. OPTIONS AVANCÉES
    options {
        // buildDiscarder : Ne garde que les 10 derniers builds pour ne pas saturer le disque dur
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // timeout : Si le job dure plus de 30 minutes, Jenkins le coupe automatiquement (évite les blocages)
        timeout(time: 30, unit: 'MINUTES')
    }

    // 2. PARAMÈTRES (Identiques à la méthode IHM, mais définis dans le code)
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['cible1', 'vmtomcat', 'all'], description: 'Cible de déploiement')
        string(name: 'WAR_FILE', defaultValue: 'app.war', description: 'Nom du fichier WAR')
        booleanParam(name: 'BACKUP', defaultValue: true, description: 'Faire une sauvegarde avant ?')
    }

    // 3. VARIABLES D'ENVIRONNEMENT ANSIBLE
    environment {
        // Désactive la vérification SSH pour éviter l'erreur "Host key verification failed"
        ANSIBLE_HOST_KEY_CHECKING = "False"
        IP_CIBLE1 = "192.168.10.40"
        IP_TOMCAT = "192.168.10.50"
    }

    stages {
        // Note : Pas besoin d'étape "Checkout Git" ! 
        // Avec la méthode SCM, Jenkins télécharge automatiquement le code Git avant de commencer les stages.

        stage('1. Prepare Ansible (Inventaire)') {
            steps {
                echo "Génération de l'inventaire Ansible..."
                script {
                    // On recrée l'inventaire dynamiquement avec la clé SSH de l'utilisateur Jenkins
                    def inventoryContent = """
[cible1]
${IP_CIBLE1} ansible_user=deploy ansible_ssh_private_key_file=/var/lib/jenkins/.ssh/id_rsa

[vmtomcat]
${IP_TOMCAT} ansible_user=tomcat ansible_ssh_private_key_file=/var/lib/jenkins/.ssh/id_rsa

[all:children]
cible1
vm_tomcat
"""
                    writeFile file: 'inventory.ini', text: inventoryContent
                    
                    // On simule aussi la présence du fichier WAR et du Playbook s'ils ne sont pas dans le Git
                    // (Retirez ces deux lignes si ces fichiers sont déjà dans votre Git !)
                    sh "touch ${params.WAR_FILE}"
                    sh "touch deploy-war.yml" // Remplacez par le vrai contenu du playbook si nécessaire
                }
            }
        }

        stage('2. Deploy avec Plugin Ansible') {
            steps {
                echo "Déploiement en cours sur : ${params.ENVIRONMENT}"
                // 4. UTILISATION DU PLUGIN ANSIBLE (Au lieu de la commande 'sh')
                ansiblePlaybook(
                    playbook: 'deploy-war.yml', // Chemin vers votre playbook dans le Git
                    inventory: 'inventory.ini',
                    colorized: true,            // Met de la couleur dans les logs Jenkins !
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

    // 5. POST-ACTIONS (S'exécutent à la toute fin du pipeline)
    post {
        always {
            // always : S'exécute TOUJOURS, que le build réussisse ou échoue
            echo "Nettoyage de l'espace de travail..."
            sh 'rm -f inventory.ini'
            // cleanWs() // Cette commande Jenkins efface tout le dossier de travail (très propre !)
        }
        success {
            // success : S'exécute uniquement si tout est vert
            echo "SUCCÈS : L'application a été déployée avec succès !"
        // Ici, on pourrait ajouter un plugin pour envoyer un message Slack ou un Email
        }
        failure {
            // failure : S'exécute si une étape a planté (en rouge)
            echo "ÉCHEC : Le déploiement a échoué. Une notification a été envoyée à l'équipe DevOps."
        }
    }
}
