#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY
    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Authorize'){
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

			println rc
        }
        stage('Test Code In Scratch Org'){
            rc = command "${toolbelt} force:org:create --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1"
                if (rc != 0) {
                    error 'Salesforce test scratch org creation failed.'
                }

            rc = command "${toolbelt} force:source:push --targetusername ciorg"
            if (rc != 0) {
                error 'Salesforce push to test scratch org failed.'
            }

            // rc = command "${toolbelt}/sfdx force:apex:test:run --targetusername ciorg --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
            // if (rc != 0) {
            //     error 'Salesforce unit test run in test scratch org failed.'
            // }

            rc = command "${toolbelt}/sfdx force:org:delete --targetusername ciorg --noprompt"
            if (rc != 0) {
                error 'Salesforce test scratch org deletion failed.'
            }

        }
        stage('Deploy Code To Developer') {
			
			if (isUnix()) {
				rmsg = sh returnStdout: true, script: "${toolbelt} force:mdapi:deploy -d manifest/. -u ${HUB_ORG}"
			}else{
				bat returnStdout: true, script: "\"${toolbelt}\" force:source:convert --rootdir force-app --outputdir tmp_convert"
				bat returnStdout: true, script: "jar -cfM newBuild.zip tmp_convert"
			   	rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:mdapi:deploy --zipfile newBuild.zip -u ${HUB_ORG}"
			}
			  
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
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
