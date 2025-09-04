stage("Publish to Nexus") {
    steps {
        script {
            // Read Maven pom.xml to get artifact info
            def pom = readMavenPom file: "pom.xml"

            // Find the artifact in the target directory with the packaging extension (e.g., jar, war)
            def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
            
            // If no artifact found, stop the pipeline with an error
            if (filesByGlob.length == 0) {
                error "No artifacts found in target directory."
            }
            
            // Take the first artifact found
            def artifact = filesByGlob[0]
            
            // Print artifact info in Jenkins console
            echo "Found artifact: ${artifact.name}, path: ${artifact.path}"
            
            // Upload artifact to Nexus using the Nexus Artifact Uploader plugin
            nexusArtifactUploader(
                nexusVersion: NEXUS_VERSION,      // Nexus version, e.g., "nexus3"
                protocol: NEXUS_PROTOCOL,          // Protocol, e.g., "http"
                nexusUrl: NEXUS_URL,               // Nexus URL (host:port)
                groupId: pom.groupId,              // Maven groupId from pom.xml
                version: pom.version,              // Maven version from pom.xml
                repository: NEXUS_REPOSITORY,      // Nexus repository name
                credentialsId: NEXUS_CREDENTIAL_ID,// Jenkins credentials ID for Nexus login
                artifacts: [
                    [
                        artifactId: pom.artifactId,  // Maven artifactId from pom.xml
                        classifier: '',              // Classifier if any (empty here)
                        file: artifact.path,         // Path to the artifact file
                        type: pom.packaging          // Packaging type (jar, war, etc.)
                    ],
                    [
                        artifactId: pom.artifactId,
                        classifier: '',
                        file: "pom.xml",             // Upload the pom.xml as well
                        type: "pom"
                    ]
                ]
            )
        }
    }
}

