#!/usr/bin/groovy

pipeline {
    agent any    
    environment {
        // This can be nexus3 or nexus2 server
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "13.232.59.140:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY_RELEASES = "maven-releases"
        // NEXUS_REPOSITORY_SNAPSHOTS = "maven-snapshots"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus_cred"
    }
	tools { 
		maven 'maven'
	}
	stages {
		stage('Code checkout') {
			steps {
				script {
					checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/ayaan2701/JAVA-CI-CD-PIPELINE.git']]])
					COMMIT = sh (script: "git rev-parse --short=10 HEAD", returnStdout: true).trim()  
                
				}
             
			}
		}
		stage('Build') {
			steps {
				sh "mvn clean package"
			}
		}
        stage('Nexus Repository') {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                                nexusVersion: NEXUS_VERSION,
                                protocol: NEXUS_PROTOCOL,
                                nexusUrl: NEXUS_URL,
                                groupId: pom.groupId,
                                version: pom.version,
                                repository: NEXUS_REPOSITORY_RELEASES,
                                credentialsId: NEXUS_CREDENTIAL_ID, 
                                artifacts: [
                                    [artifactId: pom.artifactId, 
                                     classifier: '',
                                     file: artifactPath,
                                     type: pom.packaging],
                                    [artifactId: pom.artifactId,
                                     classifier: '',
                                     file: "pom.xml", 
                                     type: "pom"]]);
                      
                    } 
                    else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}
