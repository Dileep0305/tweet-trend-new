def registry = 'https://dileepj05.jfrog.io/'
def imageName = 'dileepj05.jfrog.io/dileepj05-docker-local/ttrend'
def version   = '2.1.2'
   
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
    

    stage('Quality Gate') {
        steps {
        
            script {
                timeout(time: 1, unit: 'HOURS') {
                     def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                         }
                 }
            }
         }
         
    }

             
  stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"Jfrog"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }   

   stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }

   stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, 'Jfrog'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }
  }
 }

