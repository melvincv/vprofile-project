def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

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

        stage("Publish to Nexus") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: NEXUS_URL,
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: RELEASE_REPO,
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications'
            slackSend channel: '#jenkins-ci-cd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
