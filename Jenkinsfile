#!groovy

node {

	def SF_CONSUMER_KEY
	def SF_USERNAME
	def SF_INSTANCE_URL
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID	
    def DEPLOYDIR='src'
	def TEST_LEVEL='NoTestRun'
	def toolbelt = tool 'toolbelt'
	
	/*properties(
    [
        // other properties that you have
        pipelineTriggers([cron('H/1 * * * *')])
    ]
	)*/
	properties([pipelineTriggers([pollSCM('H/2 * * * *')])])
		
	// ------------------------------------------------------------------------
    // Check out code from source control.
    // ------------------------------------------------------------------------
		stage('checkout source') {
        checkout scm
		}

        
		
	// ------------------------------------------------------------------------
    // Select branch from repo and read values from Jenkins global variables
	// asssing values to variables.
    // ------------------------------------------------------------------------
		stage('select branch from source repository'){
			if (env.BRANCH_NAME == 'dev') {
				SF_CONSUMER_KEY=env.SF_CONSUMER_KEY_DEV
				SF_USERNAME=env.SF_USERNAME_DEV
				SF_INSTANCE_URL = env.SF_INSTANCE_URL_DEV							
			} else if (env.BRANCH_NAME == 'release') {
				SF_CONSUMER_KEY=env.SF_CONSUMER_KEY_RELEASE
				SF_USERNAME=env.SF_USERNAME_RELEASE
				SF_INSTANCE_URL = env.SF_INSTANCE_URL_DEV		
			}		
		}
		
		
		

    // ------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce.
    // JWT key credentials.
    // ------------------------------------------------------------------------

 	withEnv(["HOME=${env.WORKSPACE}"]) {	
	
	    withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {
		// --------------------------------------------------------------------
		// Authenticate to Salesforce using the server key.
		// --------------------------------------------------------------------
		
		stage('Update CLI') {
			//rc = bat returnStatus: true, script: "${toolbelt} update"
		    if (rc != 0) {
			error 'CLI update failed.'
		    }
		}
		
	// ------------------------------------------------------------------------
    // Authorize to Salesforce.
    // ------------------------------------------------------------------------
		stage('Authorize to Salesforce') {
			rc = bat returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --jwtkeyfile ${server_key_file} --username ${SF_USERNAME} --setalias dev7org"
		    if (rc != 0) {
			error 'Salesforce org authorization failed.'
		    }
		}
	
	// ------------------------------------------------------------------------
    // Get updated files from two branches.
    // ------------------------------------------------------------------------
		stage('get update files from repo') {
			//rc = bat returnStatus: true, script: "${bitbash} git diff --name-only uat master | xargs git checkout-index -f --prefix=${UATDEPLOYER}" 
			if (rc != 0) {
			error 'Salesforce org authorization failed.'
		    }
		}
	
	
	// ------------------------------------------------------------------------
    // Run apex test classes.
    // ------------------------------------------------------------------------
		stage('Running apex test classes') {
			//rc = bat returnStatus: true, script: "${toolbelt} force:apex:test:run --targetusername dev7org --testlevel ${TEST_LEVEL}"
		    if (rc != 0) {
			error 'Salesforce apex test classes failed.'
		    }
		}
		
		
	// ------------------------------------------------------------------------
	// Convert metadata.
	// ------------------------------------------------------------------------
		stage('Convert Source to Metadata') {
		   // rc = bat returnStatus: true, script: "${toolbelt} force:source:convert -p uatdeploy --outputdir ${DEPLOYDIR}"
		    if (rc != 0) {
			error 'Salesforce convert source to metadata run failed.'
		    }
		}
		
	// ------------------------------------------------------------------------
	// How to run a check-only deploy.
	// ------------------------------------------------------------------------
		stage('Check Only Deploy') {
		   //rc = command "${toolbelt} force:mdapi:deploy --checkonly --wait 10 --deploydir ${DEPLOYDIR} --targetusername dev7org"
		   if (rc != 0) {
		       error 'Salesforce deploy failed.'
		    }
		}
				
	    }
	}
}

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
		return bat(returnStatus: true, script: script);
    }
}