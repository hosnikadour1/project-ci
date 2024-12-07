pipeline {
    
	agent any
	
	tools {
        maven "MAVEN3"
        jdk   "OracleJDK8"
    }
	
     environment {
       SNAP_REPO = "vprofile-snapshot"
       NEXUS_USER = "admin"
       NEXUS_PASS = "admin"
       RELEASE_REPO = "vprofile-release"
       CENTRAL_REPO = "vpro-maven-central"
       NEXUSIP = "172.31.40.58"
       NEXUSPORT = "8081"
       NEXUS_GRP_REPO = "vpro-maven-group"
       NEXUS_LOGIN = "nexuslogin"
       SONARSERVER = "sonarserver"
       SONARSCANNER = "sonnarscanner"

    }
	
    stages{
        stage('BUILD'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
           post {
            success {
               echo "Now archiving"
               archiveArtifacts artifacts: '**/*.war'

            }

           }
        }
        stage ('TEST'){

            steps {
               sh 'mvn -s settings.xml test'

            }
        }
        stage ('checkstyle analysis'){
              steps {
            sh 'mvn -s settings.xml checkstyle:checkstyle'
        }


}
       stage ('Sonar analysis'){
        environment {
             scannerHomme = tool "${SONARSCANNER}"

        }
         steps {
                withSonarQubeEnv("${SONARSERVER}") {
             
               sh '''${scannerHomme}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

         }


       }
       stage ('quality gate'){
        steps {
            timeout (time: 1, unit: 'HOURS' ) {
                 waitForQualityGate abortPipeline: true
            }
        }

       }
       stage("upload artifact"){
           steps {
           nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        groupId: 'QA',
        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        repository: "${RELEASE_REPO}",
        credentialsId: "${NEXUS_LOGIN}",
        artifacts: [
            [artifactId: 'vproapp',
             classifier: '',
             file: 'target/vprofile-v2.war',
             type: 'jar']
        ]
     )
           }


      }
       
}
post {
   always {
    echo 'slack notification.'
    slackSend channel: "#jenkins-ci",
    color: COLOR_MAP[currentBuild.currentResult],
    message: "*${currentBuild.currentResult}:*Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"  

   }

}
}
