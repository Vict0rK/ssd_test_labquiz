// pipeline {
// 	agent none
// 	stages {
		



// 		stage('Integration UI Test') {
// 			parallel {
// 				stage('Deploy') {
// 					agent any
// 					steps {
// 						sh './jenkins/scripts/deploy.sh'
// 						input message: 'Finished using the web site? (Click "Proceed" to continue)'
// 						sh './jenkins/scripts/kill.sh'
// 					}
// 				}
// 				stage('Headless Browser Test') {
// 					agent {
// 						docker {
// 						       image 'maven' 
// 						       args '-u root' 
// 						}
// 					}
// 					steps {
// 						sh 'mvn -B -DskipTests clean package'
// 						sh 'mvn test'
// 					}
// 					post {
// 						always {
// 							junit 'target/surefire-reports/*.xml'
// 						}
// 					}
// 				}
// 			}
// 		}
// 	}
// }

pipeline {
    agent none

    stages {
        stage('Build and Test PHP') {
            agent {
                docker {
                    image 'composer:latest'
                }
            }
            stages {
                stage('Install Dependencies') {
                    steps {
                        sh 'composer install'
                    }
                }
                stage('Run PHP Unit Tests') {
                    steps {
                        sh './vendor/bin/phpunit --log-junit logs/unitreport.xml -c tests/phpunit.xml tests'
                    }
                    post {
                        always {
                            junit testResults: 'logs/unitreport.xml'
                        }
                    }
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
                    stages {
                        stage('Build and Test with Maven') {
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
    }
}

