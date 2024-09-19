pipeline{
    agent any
    stages{
        stage("Checkout"){
            steps{
                echo "Realizando el SCM"
                checkout scm
            }
        }
        stage("Compile"){
            steps{
                echo "Realizando el Build"
                script{
                    echo "*** MAVEN BUILD ***"
                    sh "mvn -B -DskipTests clean package"
                }
            }
        }
        stage("Ejecucion en paralelo"){
            parallel{
                stage("Unit Test"){
                    steps{
                        script{
                            echo "*** UNIT TESTING ***"
                            sh "mvn -B -q package"
                        }
                    }
                    post{
                        success{
                            junit 'target/surefire-reports/**/*.xml'
                        }
                    }
                }
                stage("Code Coverage"){
                    steps{
                        jacoco(
                            execPattern: 'target/**/*.exec',
                            classPattern: 'target/classes',
                            sourcePattern: 'src',
                            exclusionPattern: '**/*Test*.class'
                        )
                        publishHTML([allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: false,
                            reportDir: 'target/jacoco-report/',
                            reportFiles: 'index.html',
                            reportName: 'Code Coverage Report',
                            reportTitles: 'Code Coverage Report'
                        ])
                        junit(
                            allowEmptyResults: true,
                            testResults: '**/TEST-*.xml'
                        )
                    }
                }
                stage("Static Test"){
                    environment{
                        def scanner_home = tool 'sonar-scanner'
                        //def sonar_key = readMavenPom().getName()
                        //groupId = readMavenPom().getGroupId()
                        //artifactId = readMavenPom().getArtifactId()
                    }
                    steps{
                        script{
                            withSonarQubeEnv('sonarqube') {
                                sh "${scanner_home}/bin/sonar-scanner \
                                -Dsonar.projectKey=projectsonar1 \
                                -Dsonar.projectName=projectsonar1 \
                                -Dsonar.languaje=java \
                                -Dsonar.sources=src/ \
                                -Dsonar.java.binaries=target/classes/ \
                                -Dsonar.exclusions=src/test/ \
                                -Dsonar.host.url=http://sonarqube:9000 \
                                -Dsonar.login=admin \
                                -Dsonar.password=sonarpass"
                            }
                        }
                    }
                }
            }    
        }
        stage("Report"){
            steps{
                echo "Realizando reporte de resultados"
            }
        }
    }
}