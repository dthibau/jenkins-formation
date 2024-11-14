def dataCenters
def integrationUrl

pipeline {
   agent none 
    tools {
        jdk 'JDK17'
        maven 'MAVEN3'
    }


    stages {
        stage('Compile et tests') {
            agent any
            steps {
                echo 'Unit test et packaging'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                dir('application/target') {
                    stash includes: '*.jar', name: 'app'
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'                }
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', followSymlinks: false
                }
                unsuccessful {
                    mail bcc: '', body: 'Please review', cc: '', from: '', replyTo: '', subject: 'Build failed', to: 'david.thibau@sparks.com'
                }
            }
        }
        /*
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh 'mvn -DskipTests verify'
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN')
                    }
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }*/

        stage('Read conf') {
            agent none
            steps {
                echo "Lecture des dataCenters"
                script {
                    def jsonData = readJSON file: './deployment.json'
                    integrationUrl = jsonData['integrationURL'];
                    dataCenters = jsonData['dataCenters']
                }
            }
        }

        stage('Validation déploiement') {
/*            when {
                branch 'main'
                beforeOptions true
                beforeInput true
                beforeAgent true
            } */
            agent none
            input {
                message "Voulez vous déployer vers les datacenters $dataCenters ?"
                ok 'Déployer'
            }
            steps {
                echo "Déploiement intégration validé"
            }
        }

        stage('Déploiement intégration') {
/*            when {
                branch 'main'
                beforeOptions true
                beforeInput true
                beforeAgent true
            } */
            agent any
            steps {
                echo "Déploiement intégration vers les data centers"
                unstash 'app'
                script {
                    for ( dataCenter in dataCenters ) {
                        sh "cp *.jar $integrationUrl/${dataCenter}.jar"
                    }
                }
            }
        }

     }
    
}

