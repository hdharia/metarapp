stage 'Build and Publish'
node(){

    // COMPILE AND JUNIT
    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/hdharia/metarapp.git']]])

    ensureMaven()
    sh 'mvn clean install'
    stash includes: 'deployments/ROOT.war', name: 'war'
    step $class: 'hudson.tasks.junit.JUnitResultArchiver', testResults: 'target/surefire-reports/*.xml'
    echo "INFO - Ending build phase"
    
     docker.withServer('unix:///var/run/docker.sock'){

        def metarappImage = docker.build "hdharia/metarapp-wildfy-app:latest"
       
        sh "docker -v"
        //use withDockerRegistry to make sure we are logged in to docker hub registry
        withDockerRegistry(registry: [credentialsId: 'docker-hub-hdharia17']) { 
          metarappImage.push()
        }
   }
}

checkpoint "Build Complete"

stage 'Dev Deploy via Ansible'
node()
{
	echo "Deploying to Dev"
	unstash 'war'
	
	//Add ansible call here
	
	echo "Deployed to Dev"
}

checkpoint "Deployed to Dev"

stage "Approval for QA Deploy"
timeout(time: 10, unit: 'MINUTES')
{
   try
   { 
      input message: 'Deploy to QA?'
   } 
   catch(Exception e)
   {
      echo "No input provided, resuming build"
   } 
}

stage 'QA Deploy via Ansible'
node()
{
	echo "Deploying to QA"
	
	//Add anisble call here for QA environment
	
	echo "Deployed to QA"
}

checkpoint "Deployed to QA"

stage 'Approval for Staging Deploy'
timeout(time: 60, unit: 'SECONDS')
{
   try
   {
    input message: "Deploy to Staging?"
   } 
   catch(Exception e)
   {
      echo "No input provided, resuming build"
   } 
}

stage 'Staging Deploy'
node()
{
	echo "Deploying to Staging"
	
	//Hook in openshift deployment
	
	echo "Deployed to Staging"
}

checkpoint "Deployed to Staging"
stage 'Approval for Production Deploy'
timeout(time: 60, unit: 'SECONDS')
{
   input message: "Deploy to Prod?"
}

stage 'Deploy to Production'
node()
{
	echo "Deploying to Prod"
	
	//Hook into oepnshift deployment
	
	echo "Deployed to Prod"
}

/**
 * Deploy Maven on the slave if needed and add it to the path
 */
def ensureMaven() {
    env.PATH = "${tool 'mvn'}/bin:${env.PATH}"
}