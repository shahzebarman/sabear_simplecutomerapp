pipeline {
    agent any
    tools {
        maven "maven" // Replace with actual Maven tool name in Jenkins if different
    }
    environment {
        // Nexus repository settings
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.226.153.211:8081"
        NEXUS_REPOSITORY = "new-nexus-repo"
        NEXUS_CREDENTIAL_ID = "Nexus"

        // Sonar scanner tool
        SCANNER_HOME = tool 'sonar-scanner'

        // This must match the name of your SonarQube installation in Jenkins > Configure System
        SONARQUBE_INSTALLATION_NAME = "sonar-qube-server"
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
                withSonarQubeEnv("${env.SONARQUBE_INSTALLATION_NAME}") {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=2.0 \
                        -Dsonar.sources=src \
                        -Dsonar.binaries=target/classes \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.jacoco.reportPath=target/jacoco.exec
                    """
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    if (filesByGlob.length == 0) {
                        error "No artifacts found in target directory."
                    }
                    def artifact = filesByGlob[0]
                    echo "Found artifact: ${artifact.name}, path: ${artifact.path}"

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
                             file: artifact.path,
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
    }

    post {
        success {
            echo '✅ Build, SonarQube analysis, and Nexus deployment succeeded!'
        }
        failure {
            echo '❌ Something went wrong during build, analysis, or deployment.'
        }
    }
}
