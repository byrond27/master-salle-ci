pipeline {
      agent any

   stages {
      stage("Git Checkout") {
         steps {
            echo "Git checkout"
            git "https://github.com/byrond27/master-salle-ci"
         }
      }
      stage("Cleaning Package and Compile") {
         steps {
            echo "Cleaning Package"
            sh script: "mvn clean package -DskipTests"
            echo "Compile the package"
            sh label:"", script: "mvn compile"
         }
      }
      stage ("Sonarqube"){
         sh script:"mvn sonar:sonar \
                  -Dsonar.projectKey=Jenkins-Java \
                  -Dsonar.host.url=http://localhost:9000 \
                  -Dsonar.login=d41a669c82aacf62ab62c16255256bdb326b3596"
      }
      stage ("Tests") {
          steps {
            sh script: "mvn test"
            junit "target/surefire-reports/*.xml"
          }
      }
      stage("Acceptance Test"){
         steps{
            echo "Executing Acceptance Test"
            sh script: "mvn verify"
         }
      }
      stage("Package") {
         steps {
            echo "Package version ${new_version}"
            sh script: "mvn package -DskipTests"
         }
      }
     stage("Build Docker Image") {
         steps {
            sh "docker build -f src/main/docker/Dockerfile.jvm -t quarkus/code-with-quarkus-jvm ."
         }
     }
     stage("Produccion") {
        input {
            message "Pasamos a producción?"
        }
        steps {
            echo "Production"
            sh "docker stop maven_${old_version} || (exit 0)"
            sh "docker run --name maven_${new_version} -i --rm -p 8082:8080 -d quarkus/code-with-quarkus-jvm"
        }
     }
   }
   post {  
      always {  
         mail bcc: "", body: "<b>Pipeline Finished</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: "", charset: "UTF-8", from: "", mimeType: "text/html", replyTo: "", subject: "Pipeline FINISHED CI: Project name -> ${env.JOB_NAME}", to: "${notification_email}";  
      }
      success {  
         mail bcc: "", body: "<b>Pipeline Success</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: "", charset: "UTF-8", from: "", mimeType: "text/html", replyTo: "", subject: "Pipeline SUCCESS CI: Project name -> ${env.JOB_NAME}", to: "${notification_email}";  
      }  
      failure {  
         mail bcc: "", body: "<b>Pipeline Failure</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: "", charset: "UTF-8", from: "", mimeType: "text/html", replyTo: "", subject: "Pipeline ERROR CI: Project name -> ${env.JOB_NAME}", to: "${notification_email}";  
      }  
   }  
}
