pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
}
    stages {
        stage('build') {
            steps {
                echo "------- Build Started -------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "------- Build Completed -------"
            }
        }

        stage("test"){
            steps{
                echo "---- Unit Test Started ----"
                sh "mvn surefire-report:report"
                echo "----- Unit Test Completed -----"
            }
        }

        stage('SonarQube analysis') {
    environment {
      scannerHome = tool 'Sonar-scanner'
    }
    steps{
    withSonarQubeEnv('sonarcloud-server') { // If you have configured more than one global server connection, you can specify its name
      sh "${scannerHome}/bin/sonar-scanner"
    }
    }
  }
    }

    stage("Quality Gate"){
        steps{
            script{
                timeout(time: 1, unit: 'Hours'){
                    def qg = waitForQualityGate()
                    if (qg.status! = 'ok'){
                        error "Pipeline aborted due to Quality Gate Failure: ${qg.status}"
                    }
                }
            }
        }
    }
}

