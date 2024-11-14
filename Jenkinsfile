def props = readJSON file: 'parameters.json'
pipeline {
    agent none 
    tools {
        maven 'Maven'
        jdk 'JDK17'

    }

    stages {
        stage('Build and test') {
            agent any
            steps {
                echo '============  Build'
                sh 'mvn -Dmaven.test.failure.ignore clean package'
            }
            post {
                always { 
                    echo '============  Publish'
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
                success {  
                    echo '============  Archive'  
                    archiveArtifacts '**/target/*.jar'
                }
                failure {
                    mail bcc: '', body: 'Ca ne fonctionne pas', cc: '', from: '', replyTo: '', subject: 'package failed', to: 'stageojen@plbformation.com'
                }  
            } 
        }
        stage('Analyse dependance') {            
            parallel {
                stage('Dependances') {
                    agent any
                    steps {
                        echo '=========== Analyse des dépendances du projet'
                        sh 'mvn -DskipTests verify'
                    }                 
                }
                
                stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('Sonarqube')
                    }
                    steps {
                        echo '=========== Analyse sonar'
                    //    sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                    }  
                }
                
            }      
        }  
        stage('Deploy') {
            agent any
            when { 
                beforeInput true
                beforeAgent true
                branch 'master'
            } 
            input {
                message 'Dans quel Data Center, voulez-vous déployer l’artefact ?'
                parameters {
                    choice choices: ['Paris ', 'Lille ', 'Lyon'], name: 'DC'
                }
            }
            steps {
                echo "============ Déploiement intégration ${DC}"
                stash name: 'build_result', includes: '**/target/*.jar'
                dir("/home/plb/${DC}") {
                    unstash 'build_result'
                } 
            }
        }
    }
}

