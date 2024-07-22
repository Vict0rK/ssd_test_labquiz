pipeline {
    agent none

    stages {
        stage('OWASP Dependency-Check Vulnerabilities Lab 6') {
            agent any
            steps {
                dependencyCheck additionalArguments: '''
                    -o './'
                    -s './'
                    -f 'ALL'
                    --prettyPrint''', 
                odcInstallation: 'OWASP Dependency Check'
                
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
		
		stage('Next Generation Warning Plugin Lab 7b') {
			agent any 
			steps{
				git branch: 'main', url: 'https://github.com/Vict0rK/ssd_test_labquiz.git'
				sh '/var/jenkins_home/apache-maven-3.9.8/bin/mvn --batch-mode -V -U -e clean verify -Dsurefire.useFile=false -Dmaven.test.failure.ignore'
				sh '/var/jenkins_home/apache-maven-3.9.8/bin/mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd findbugs:findbugs'

			}
			post {
				always {
				junit testResults: '**/target/surefire-reports/TEST-*.xml'

				recordIssues(enabledForFailure: true, tools: [
					mavenConsole(),
					java(),
					javaDoc()
				])
				recordIssues(enabledForFailure: true, tool: checkStyle())
				recordIssues(enabledForFailure: true, tool: spotBugs(pattern: '**/target/findbugsXml.xml'))
				recordIssues(enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml'))
				recordIssues(enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml'))
				}
			}
		}

        stage('Integration UI Test') {
            parallel {
                stage('Deploy') {
                    agent any
                    steps {
                        sh './jenkins/scripts/deploy.sh'
                        input message: 'Finished using the web site? (Click "Proceed" to continue)'
                        sh './jenkins/scripts/kill.sh'
                    }
                }

                stage('Headless Browser Test') {
                    agent {
                        docker {
                            image 'maven'
                            args '-u root'
                        }
                    }
                    steps {
                        sh 'mvn -B -DskipTests clean package'
                        sh 'mvn test'
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }
            }
        }
    }
}
