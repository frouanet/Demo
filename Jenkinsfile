def props
def mybd
def mydcs
def level

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
                    stash name: 'build_result', includes: '**/target/*.jar'
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
                    //    sh 'mvn -DskipTests verify'
                        sleep 3
                    }                 
                }
                
                stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('Sonarqube')
                    }
                    steps {
                        echo '=========== Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                        checkSonarQualityGate()
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
                dir("/home/plb/${DC}") {
                    unstash 'build_result'
                } 
            }
        }
        
        stage('Read parms'){
            agent any       
            steps{
                echo "============ Read "
                script{
                    def myjsdata = GetMyDC()
                    mybd = myjsdata['bdir']
                    mydcs = myjsdata['dcs']
                    level = myjsdata['level']
                }  
                
            }
        }
        stage('May I deploy'){
            agent none
            steps{ 
                input (
                    message: "Pret pour deployer sur ${mydcs} ?",
                    ok: 'deployer'
                )
            } 
        }  
        stage('Deploy Second method'){
            agent any
            steps{
                script{
                    for (int i = 0; i < mydcs.size(); ++i) {
                        sh "echo =============== building ${mydcs[i]} to ${mybd}"
                        dir("${mybd}/${mydcs[i]}") {
                            unstash 'build_result'
                        }
                    } 
                } 

            }  
        } 
    }
}

def GetMyDC(){
    def props = readJSON file: './parameters.json'
    def mybuildir = props['builddir'] 
    def mydcs = props['datacenters']
    def mylevel = props['level']  
    return ['dcs' : mydcs, 'bdir' : mybuildir, 'level' : mylevel] 
}


def checkSonarQualityGate(){
    // Get properties from report file to call SonarQube 
    def sonarReportProps = readProperties  file: 'target/sonar/report-task.txt'
    def sonarServerUrl = sonarReportProps['serverUrl']
    def ceTaskUrl = sonarReportProps['ceTaskUrl']
    def ceTask

    // Get task informations to get the status
    timeout(time: 4, unit: 'MINUTES') {
        waitUntil {
            withCredentials ([string(credentialsId: 'Sonarqube', variable : 'token')]) {
                def response = sh(script: "curl -u ${token}: ${ceTaskUrl}", returnStdout: true).trim()
                ceTask = readJSON text: response
            }

            echo ceTask.toString()
              return "SUCCESS".equals(ceTask['task']['status'])
        }
    }

    // Get project analysis informations to check the status
    def ceTaskAnalysisId = ceTask['task']['analysisId']
    def qualitygate

    withCredentials ([string(credentialsId: 'Sonarqube', variable : 'token')]) {
        def response = sh(script: "curl -u ${token}: ${sonarServerUrl}/api/qualitygates/project_status?analysisId=${ceTaskAnalysisId}", returnStdout: true).trim()
        qualitygate =  readJSON text: response
    }

    echo qualitygate.toString()
    // when debug
    if (level.equals("debug")){
        qualitygate['projectStatus']['status'] = "ERROR"
    } 
    
    if ("ERROR".equals(qualitygate['projectStatus']['status'])) {
        unstable "Quality Gate failure"
    }
}


