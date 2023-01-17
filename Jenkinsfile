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
             withSonarQubeEnv('sonar-pro') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

         }


       }

}
}