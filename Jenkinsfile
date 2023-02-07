pipeline {
    
	agent any
	tools {
        maven "Maven3"
        jdk "OpenJDK8"
    }

    environment {
        SNAP_REPO = "vprofile-snapshot"
        NEXUS_CRED = credentials('nexuslogin')
        NEXUS_USER = '$NEXUS_CRED_USR'
        NEXUS_PASS = '$NEXUS_CRED_PSW'
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vprofile-maven-central"
        NEXUS_GRP_REPO = "vprofile-maven-group"
        NEXUS_URL = "172.31.10.59:8081"
        ARTVERSION = "${env.BUILD_ID}"
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn -s settings.xml clean install -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
                steps {
                    sh 'mvn test'
                }
            }

	// stage('INTEGRATION TEST'){
    //         steps {
    //             sh 'mvn verify -DskipUnitTests'
    //         }
    //     }
		
        stage ('CODE ANALYSIS: CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('CODE ANALYSIS: SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonarscanner'
          }

          steps {
            withSonarQubeEnv('sonar-svr') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
          }
        }

    //     stage("Publish to Nexus Repository Manager") {
    //         steps {
    //             script {
    //                 pom = readMavenPom file: "pom.xml";
    //                 filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
    //                 echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
    //                 artifactPath = filesByGlob[0].path;
    //                 artifactExists = fileExists artifactPath;
    //                 if(artifactExists) {
    //                     echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
    //                     nexusArtifactUploader(
    //                         nexusVersion: NEXUS_VERSION,
    //                         protocol: NEXUS_PROTOCOL,
    //                         nexusUrl: NEXUS_URL,
    //                         groupId: NEXUS_REPOGRP_ID,
    //                         version: ARTVERSION,
    //                         repository: NEXUS_REPOSITORY,
    //                         credentialsId: NEXUS_CREDENTIAL_ID,
    //                         artifacts: [
    //                             [artifactId: pom.artifactId,
    //                             classifier: '',
    //                             file: artifactPath,
    //                             type: pom.packaging],
    //                             [artifactId: pom.artifactId,
    //                             classifier: '',
    //                             file: "pom.xml",
    //                             type: "pom"]
    //                         ]
    //                     );
    //                 } 
	// 	    else {
    //                     error "*** File: ${artifactPath}, could not be found";
    //                 }
    //             }
    //         }
    //     }


    }


}
