pipeline {
    agent {
        label 'Backend'
    }
    
    tools {
        maven "MAVEN"
    }
    
    environment {
        TOMCAT_HOME = '/opt/tomcat'
        WAR_FILE = 'Ehealth-B-0.0.1-SNAPSHOT.war'
        TOMCAT_WEBAPPS = "${TOMCAT_HOME}/webapps"
        APP_CONTEXT = 'ehealth.war'
        TOMCAT_PORT = '8086'
    }
    
    stages {
        stage('Cloner le repo') {
            steps {
                echo 'Clonage du repository...'
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: '72145410-c633-4919-8bdd-f5ad0b33e759',
                        url: 'https://github.com/E-Health-Organization/E-health-Backend.git'
                    ]]
                )
            }
        }
        
        stage('Build avec Maven') {
            steps {
                echo 'Compilation du projet dans Backend/Ehealth-B...'
                sh '''
                    cd Backend/Ehealth-B
                    mvn clean package -DskipTests || exit 1
                '''
            }
        }
        
        stage('Vérifier la structure des fichiers') {
            steps {
                echo 'Vérification des fichiers générés dans Backend/Ehealth-B...'
                sh 'ls -R Backend/Ehealth-B/target'
            }
        }
        
        stage('Archiver les artefacts') {
            steps {
                echo 'Archivage du fichier WAR...'
                archiveArtifacts artifacts: 'Backend/Ehealth-B/target/*.war', fingerprint: true
            }
        }

        stage('Configurer Tomcat') {
            steps {
                echo 'Redémarrage de Tomcat...'
                sh '''
                    sudo systemctl restart tomcat || exit 1
                '''
            }
        }

        stage('Déployer sur Tomcat') {
            steps {
                echo 'Déploiement du fichier WAR sur Tomcat...'
                script {
                    // Vérifier que le fichier WAR existe
                    if (!fileExists("${WAR_FILE}")) {
                        error "Le fichier WAR ${WAR_FILE} n'existe pas. Le déploiement est annulé."
                    }
                    
                    sh """
                        sudo cp ${WAR_FILE} ${TOMCAT_WEBAPPS}/${APP_CONTEXT} || exit 1
                        sudo systemctl restart tomcat || exit 1
                    """
                }
            }
        }

        stage('Vérification de l’API') {
            steps {
                echo 'Test de l’API après déploiement...'
                sh """
                    sleep 10
                    until curl -X GET http://localhost:${TOMCAT_PORT}/ehealth/api/patients; do
                        echo "En attente du démarrage de l'application..."
                        sleep 5
                    done
                """
            }
        }
    }
    
    post {
        success {
            echo '✅ Build et déploiement réussis sur Tomcat (port 8086!)'
        }
        failure {
            echo '❌ Le build ou le déploiement a échoué.'
        }
    }
}
