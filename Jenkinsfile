pipeline {
    agent any

    tools {
        maven "maven"
    }

    environment {
        // Nexus configuration
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.226.153.211:8081"
        NEXUS_REPOSITORY = "maven-releases" // Make sure this is correct in Nexus
        NEXUS_CREDENTIAL_ID = "Nexus"

        // SonarQube configuration
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage("Clone Code") {
            steps {
                git 'https://github.com/shahzebarman/sabear_simplecutomerapp.git'
            }
        }

        stage("Build with Maven") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

       stage("SonarQube Analysis") {
    steps {
        withSonarQubeEnv("${env.SONARQUBE_INSTALLATION_NAME}") {
            sh """
                echo "⚙️ Checking if compiled classes exist"
                ls -la target/classes || echo "⚠️ target/classes does not exist!"

                ${SCANNER_HOME}/bin/sonar-scanner \
                -Dsonar.projectKey=Ncodeit \
                -Dsonar.projectName=Ncodeit \
                -Dsonar.projectVersion=2.0 \
                -Dsonar.sources=src \
                -Dsonar.java.binaries=target/classes \
                -Dsonar.junit.reportsPath=target/surefire-reports \
                -Dsonar.jacoco.reportPath=target/jacoco.exec \
                -X
                    """
                
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    def artifactPath = filesByGlob[0].path
                    def artifactExists = fileExists artifactPath

                    if (artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        
                        nexusArtifactUploader(
                            nexusVersion: env.NEXUS_VERSION,
                            protocol: env.NEXUS_PROTOCOL,
                            nexusUrl: env.NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: env.NEXUS_REPOSITORY,
                            credentialsId: env.NEXUS_CREDENTIAL_ID,
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
                    } else {
                        error "*** File: ${artifactPath} could not be found"
                    }
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
}
