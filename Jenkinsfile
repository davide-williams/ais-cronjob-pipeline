pipeline {
  agent {
     label 'OCPNonProdAgent'
   }  
  environment{
      NAMESPACE = 'ais-service-demo'
      APP_NAME = 'ais-cronjob-demo'      
      EMAIL_RECIPENT ='DavidE.Williams@vita.virginia.gov'
      APP_BUILD_PATH = "${WORKSPACE}/helloworld-executable-jar"
      APP_IMAGE = "image-registry.openshift-image-registry.svc:5000/${NAMESPACE}/${APP_NAME}"
   }
  stages {
    
    stage('Build Java Application') {
      steps{
        script{
         dir("${APP_BUILD_PATH}"){         
           sh "mvn --version"
         
           echo "--- Update maven POM.xml"
           def pom = readMavenPom file: 'pom.xml'
           echo "pom->version: " +pom.version
           pom.version = env.BUILD_NUMBER
           echo "POM.xml after change"
           echo "pom->version: " +pom.version
           writeMavenPom model: pom
           
           sh "mvn clean package"
         }
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        dir("${APP_BUILD_PATH}"){ 
          echo 'Building image'    
          sh "podman build --no-cache -f Dockerfile --tag ${APP_IMAGE}:"+env.BUILD_NUMBER
        }
      }
    }
    
     stage('Authenticate to Internal OCP Registry'){
        steps{
            echo 'Authenticating to Openshift Registry'
            sh 'podman login -u jenkins -p $(oc whoami -t) image-registry.openshift-image-registry.svc:5000 --tls-verify=false'
        }
    }
    stage('Push to Internal OCP Registry') {
      steps {
        echo 'Pushing image'
        sh "podman push ${APP_IMAGE}:"+env.BUILD_NUMBER+ " --tls-verify=false"
      }
    }
    
     stage('Build OCP application') {
      steps{
        script{
         dir("${WORKSPACE}"){ 
           
           def deploymentYaml = readYaml file: "cron-deployment.yaml"
           
           echo "deployment yaml: " + deploymentYaml
           echo "image: "+deploymentYaml.spec.jobTemplate.spec.template.spec.containers[0].image
           deploymentYaml.spec.jobTemplate.spec.template.spec.containers[0].image = "${APP_IMAGE}:"+env.BUILD_NUMBER
           echo "image after update: "+deploymentYaml.spec.jobTemplate.spec.template.spec.containers[0].image
           
           writeYaml file: "cron-deployment.yaml", data: deploymentYaml, overwrite: true
         }
        }
      }
    }
    
      stage('Deploy OCP application') {
       steps {
       dir("${WORKSPACE}"){ 
         echo '---rolling out application'
	     sh "oc apply -f cron-deployment.yaml"
       }
      }
    }
    
 }
  post {
         // Clean after build
         always {
            echo 'Cleaning up after build...'
             cleanWs(cleanWhenNotBuilt: false,
                     deleteDirs: true,
                     disableDeferredWipeout: true,
                     notFailBuild: true,
                     patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                                [pattern: '.propsfile', type: 'EXCLUDE']])
         }
         success {  
             echo 'Sending successful build message'              
             mail body: "<b>BUILD SUCCESS</b><br><br>Project: ${env.JOB_NAME}<br><br>Build Number: ${env.BUILD_NUMBER} <br><br> URL of build: <a href=\"${env.BUILD_URL}\">${env.BUILD_URL}</a>", charset: 'UTF-8', from: 'ais-jenkis<no-reply@cov.virginia.gov>', mimeType: 'text/html', subject: "Jenkins BUILD SUCCESS: ${env.JOB_NAME}", to: "${EMAIL_RECIPENT}";
         }  
         failure {  
             echo 'Sending failure build message'  
             mail body: "<b>BUILD FAILURE</b><br><br>Project: ${env.JOB_NAME}<br><br>Build Number: ${env.BUILD_NUMBER} <br><br> URL of build: <a href=\"${env.BUILD_URL}\">${env.BUILD_URL}</a>", charset: 'UTF-8', from: 'ais-jenkis<no-reply@cov.virginia.gov>', mimeType: 'text/html', subject: "Jenkins BUILD FAILURE: ${env.JOB_NAME}", to: "${EMAIL_RECIPENT}";
         }   
     }
  
  
 }
   
   