pipeline {
    agent any
    tools {
        maven "maven"
    }

    environment {
        // Nexus details
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.226.153.211:8081"
        NEXUS_REPOSITORY = "Sunil"
        NEXUS_CREDENTIAL_ID = "Nexus"

        // Sonar Scanner
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage("Clone Code") {
            steps {
                git 'https://github.com/shahzebarman/sabear_simplecutomerapp.git'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec
                    '''
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

                    if (filesByGlob.length == 0) {
                        error "*** No artifacts found to upload."
                    }

                    def artifactPath = filesByGlob[0].path
                    echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: artifactPath,
                             type: pom.packaging],
                            [artifactId: pom.artifactId,
                             classifier: '',
                             file: "pom.xml",
                             type: "pom"]
                        ]
                    )
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat_credential', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    script {
                        def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                        echo "Deploying ${warFile} to Tomcat at context path /simplecustomerapp ..."
                        sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS \
                            -T ${warFile} \
                            "http://52.87.164.24:8080/manager/text/deploy?path=/simplecustomerapp&update=true"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}
