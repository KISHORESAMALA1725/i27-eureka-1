pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
     environment {
        APPLICATION_NAME = "eureka"
     }
     stages {
        stage ('***** Build Stage *****') {
            steps {
                sh 'mvn clean package -DskipTest=true'
                archiveArtifacts 'target/*.jar'
            }
        }

        stage ('sonarqube'){
           steps {
               echo "*******Starting Sonar Scans with Quality Gates*********"
               withSonarQubeEnv('sonarqube') {// SonarQube is the name we configured in Manage Jenkins > system > Sonarqube , it hsould match exactly
                   sh """
                      mvn sonar:sonar \
                        -Dsonar.projectKey=i27-eureka \
                        -Dsonar.host.url=http://34.86.131.81:9000 \
                        -Dsonar.login=sqa_7c6ad449db3d7d80022780dcc34d31da846ee528
                      """
                 }
                   timeout (time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
               }
             
           }
        }
     }
}
