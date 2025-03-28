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
        stage ('Build Stage') {
            steps {
                echo "Checking Maven installation..."
                sh 'which mvn'  // Check where mvn is located
                sh 'mvn -v'      // Print Maven version
                sh 'mvn clean package -DskipTests=true'
                archiveArtifacts 'target/*.jar'
            }
        }

        stage ('SonarQube Scan') {
            steps {
                echo "******* Starting SonarQube Scans with Quality Gates *********"
                withSonarQubeEnv('sonarqube') { 
                    sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=i27-eureka \
                      -Dsonar.host.url=http://34.86.131.81:9000 \
                      -Dsonar.login=sqa_7c6ad449db3d7d80022780dcc34d31da846ee528
                    """
                }
                timeout(time: 5, unit: 'MINUTES') { // Increased timeout for SonarQube scan
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
