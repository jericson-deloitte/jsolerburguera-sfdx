#!groovy
 
import groovy.json.JsonSlurperClassic

node {


    env.SF_CONSUMER_KEY = "3MVG9JEx.BE6yifNYKyCrEC8c2Wz82eH0hupqiv3hGKpfBZ_n3GBnV2A3QKVSOHCha0g..FgP2qouVjsZMSEo"
    env.SF_INSTANCE_URL = $LOGIN_URL
 println $LOGIN_URL 
    env.SF_USERNAME = "jsolerburguera@dev-sfdx.com"
    env.SERVER_KEY_CREDENTALS_ID = "b30e00f9-2470-4551-957e-bcbf3b82a060" 
    env.SPRINTST_USERNAME = "jsolerburguera@dev-sfdx.com"

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTALS_ID=env.SERVER_KEY_CREDENTALS_ID
    def TEST_LEVEL='RunLocalTests'
    def PACKAGE_NAME='sales'
    def PACKAGE_VERSION
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"

    def toolbelt = tool 'toolbelt'

    //echo "WORKSPACE is ${env.WORKSPACE} or ${WORKSPACE}"

    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
    
    withEnv(["HOME=${env.WORKSPACE}"]) {
        
         withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {

            // -------------------------------------------------------------------------
            // Authorize the Dev Hub org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize DevHub') {
                rc = command "sfdx force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --setalias HubOrg"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Create new scratch org to test your code.
            // -------------------------------------------------------------------------

            stage('Create Test Scratch Org') {
                rc = command "sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce test scratch org creation failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Display test scratch org info.
            // -------------------------------------------------------------------------

            stage('Display Test Scratch Org') {
                rc = command "sfdx force:org:display --targetusername ciorg"
                if (rc != 0) {
                    error 'Salesforce test scratch org display failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Push source to test scratch org.
            // -------------------------------------------------------------------------

            stage('Push To Test Scratch Org') {
                rc = command "sfdx force:source:push --targetusername ciorg"
                if (rc != 0) {
                    error 'Salesforce push to test scratch org failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Run unit tests in test scratch org.
            // -------------------------------------------------------------------------

            /* stage('Run Tests In Test Scratch Org') {
                rc = command "sfdx force:apex:test:run --targetusername ciorg --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
                if (rc != 0) {
                    error 'Salesforce unit test run in test scratch org failed.'
                }
            } */


            // -------------------------------------------------------------------------
            // Delete test scratch org.
            // -------------------------------------------------------------------------

            stage('Delete Test Scratch Org') {
                rc = command "sfdx force:org:delete --targetusername ciorg --noprompt"
                if (rc != 0) {
                    error 'Salesforce test scratch org deletion failed.'
                }
            }


            // -------------------------------------------------------------------------
            // Create package version.
            // -------------------------------------------------------------------------

            stage('Create Package Version') {
                if (isUnix()) {
                    output = sh returnStdout: true, script: "sfdx force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername HubOrg"
                } else {
                    output = bat(returnStdout: true, script: "sfdx force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername HubOrg").trim()
                    output = output.readLines().drop(1).join(" ")
                }

                echo "The package version created is ${output}"

                // Wait 5 minutes for package replication.
                sleep 300

                def jsonSlurper = new JsonSlurperClassic()
                def response = jsonSlurper.parseText(output)

                PACKAGE_VERSION = response.result.SubscriberPackageVersionId

                response = null
            }


            // -------------------------------------------------------------------------
            // Create new scratch org to install package to.
            // -------------------------------------------------------------------------

            /* stage('Create Package Install Scratch Org') {
                rc = command "sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias installorg --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce package install scratch org creation failed.'
                }
            } */


            // -------------------------------------------------------------------------
            // Display install scratch org info.
            // -------------------------------------------------------------------------

            /* stage('Display Install Scratch Org') {
                rc = command "sfdx force:org:display --targetusername installorg"
                if (rc != 0) {
                    error 'Salesforce install scratch org display failed.'
                }
            } */


            // -------------------------------------------------------------------------
            // Install package in scratch org.
            // -------------------------------------------------------------------------

            /* stage('Install Package In Scratch Org') {
                rc = command "sfdx force:package:install --package ${PACKAGE_VERSION} --targetusername installorg --wait 10"
                if (rc != 0) {
                    error 'Salesforce package install failed.'
                }
            } */


            // -------------------------------------------------------------------------
            // Run unit tests in package install scratch org.
            // -------------------------------------------------------------------------

            /* stage('Run Tests In Package Install Scratch Org') {
                rc = command "sfdx force:apex:test:run --targetusername installorg --resultformat tap --codecoverage --testlevel ${TEST_LEVEL} --wait 10"
                if (rc != 0) {
                    error 'Salesforce unit test run in pacakge install scratch org failed.'
                }
            } */ 


            // -------------------------------------------------------------------------
            // Delete package install scratch org.
            // -------------------------------------------------------------------------

            /* stage('Delete Package Install Scratch Org') {
                rc = command "sfdx force:org:delete --targetusername installorg --noprompt"
                if (rc != 0) {
                    error 'Salesforce package install scratch org deletion failed.'
                }
            } */


            // -------------------------------------------------------------------------
            // Install package in SPRINTST Sandbox.
            // -------------------------------------------------------------------------

            stage('Install Package In SPRINTST Org') {

                //In case we want to deploy using unlocked packages
                rc = command "sfdx force:package:install --package ${PACKAGE_VERSION} --targetusername ${SPRINTST_USERNAME} --wait 10"
                if (rc != 0) {
                    error 'Salesforce package install failed in Sprintst.'
                }

                //In case we want to deploy using mdapi
                /* # Create a temp directory in your repository
                mkdir mdapi_temp_dir
                
                # Convert your sources to the metadata API format
                # and place them in the temp directory
                rc = command "sfdx force:source:convert -d mdapi_temp_dir/"
                if (rc != 0) {
                    error 'Error converting source format to mdapi format'
                }
                
                # Deploy the sources to a target org
                # Assuming that $targetOrg is a username on the target org
                # Command times out if it takes longer than 10 minutes
                rc = command "sfdx force:mdapi:deploy -d mdapi_temp_dir/ --targetusername ${SPRINTST_USERNAME} --wait 10"
                if (rc != 0) {
                    error 'Error Deploying into Sprintst'
                }
                
                # Remove the temp directory
                rm -fr mdapi_temp_dir */
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
