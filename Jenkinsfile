pipeline {
    agent any

    tools {
        maven 'Maven'
        nodejs "NodeJs"
    }
    stages {
      stage ('Initial') {
            steps {
              sh '''
                   echo "PATH = ${PATH}"
                   echo "M2_HOME = ${M2_HOME}"
               '''
            }
        }
        stage ('Compile') {
            steps {
                 sh 'mvn clean compile -e'
            }
        }
        stage ('Test') {
            steps {
                 sh 'mvn clean test -e'
            }
        }

        stage('SonarQube analysis') {
           steps{
                script {
                    def scannerHome = tool 'SonarQube Scanner';//def scannerHome = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('Sonar Server') {
                      sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Ms-Maven -Dsonar.sources=target/ -Dsonar.host.url=http://172.18.0.3:9000 -Dsonar.login=f74fc670543b5f3d217066fcdd8340ec592be0cd"
                      //sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Maven -Dsonar.host.url=http://172.18.0.3:9000 -Dsonar.login=f74fc670543b5f3d217066fcdd8340ec592be0cd -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html"

                    }
                }
           }
        }

        stage ('SCA') {
            steps {
                 sh 'mvn org.owasp:dependency-check-maven:check'
                dependencyCheckPublisher failedNewCritical: 5, failedTotalCritical: 10, pattern: 'terget/dad.xml', unstableNewCritical: 3, unstableTotalCritical: 5
            }
        }
        
         stage('ZAP'){
        			steps{
        				script{
        				    env.DOCKER = tool "Docker"
        				    env.DOCKER_EXEC = "${DOCKER}/bin/docker"
        				    env.TARGET = 'https://demo.testfire.net/'

        				    sh '${DOCKER_EXEC} rm -f zap2'
        				    sh '${DOCKER_EXEC} pull owasp/zap2docker-stable'
                            sh '${DOCKER_EXEC} run --add-host="localhost:192.168.100.4" --rm -e LC_ALL=C.UTF-8 -e LANG=C.UTF-8 --name zap2 -u zap -p 8090:8080 -d owasp/zap2docker-stable zap.sh -daemon -port 8080 -host 0.0.0.0 -config api.disablekey=true'
                            sh '${DOCKER_EXEC} run --add-host="localhost:192.168.100.4" -v /Users/asajuro/Documents/BCI/AnalyzeQAS/Jenkins-Practica/USACH/Dockerfile/zap/jenkins_home/tools:/zap/wrk/:rw --rm -i owasp/zap2docker-stable zap-baseline.py -t "https://demo.testfire.net/" -I -r zap_baseline_report2.html -l PASS'
        				}
        			}
        		}

        		stage('Publish'){
        			steps{
        				publishHTML([
        				    allowMissing: false,
        				    alwaysLinkToLastBuild: false,
        				    keepAll: false,
        				    reportDir: '/var/jenkins_home/tools',
        				    reportFiles: 'zap_baseline_report2.html',
        				    reportName: 'HTML Report',
        				    reportTitles: ''])
        				    //archiveArtifacts artifacts: '/var/jenkins_home/tools/zap_baseline_report2.html'
        			}
        		}

    }
}
