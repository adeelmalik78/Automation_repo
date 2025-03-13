pipeline {
	agent any
	
	options {
		// This is required if you want to clean before build
		skipDefaultCheckout(true)
	}
	
	environment {
		PATH="/opt/liquibase/liquibase-4.30.0/:$PATH"
		JAVA_HOME="/usr/lib/jvm/jdk-17.0.13-oracle-x64"
		// DRY_RUN = "${params.DRY_RUN}"
		// LIQUIBASE_COMMAND_URL = credentials('LIQUIBASE_DEV_URL')
		// LIQUIBASE_COMMAND_USERNAME = credentials('LIQUIBASE_USER')
		// LIQUIBASE_COMMAND_PASSWORD = credentials('LIQUIBASE_PASSWORD')
		LIQUIBASE_LICENSE_KEY = credentials('LIQUIBASE_KEY')
		// LIQUIBASE_DEFAULTS_FILE="liquibase.properties"
		// LIQUIBASE_COMMAND_CHANGELOG_FILE="../${params.LIQUIBASE_COMMAND_CHANGELOG_FILE}"
	}

	stages {
		stage('Workspace') {
			steps {
			
			    script {
				  currentBuild.displayName = "#" + env.BUILD_NUMBER
				}
			
				// Clean before build
				cleanWs() // This requires ws-cleanup plugin
					
				// We need to explicitly checkout from SCM here
				checkout scm

				// checkout automation repo
				checkout scmGit(
					branches: [[name: 'feature/developer']], 
					extensions: [], 
					userRemoteConfigs: [[url: '${GIT_REPO}']]
				)
				checkout scmGit(
					branches: [[name: '*/master']], 
					extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'dsa']], 
					userRemoteConfigs: [[credentialsId: 'amalik-bitbucket', 
					url: 'http://amalik@ca.liquibase.net:7990/scm/ccb/dsa_spring_demo.git']]
				)

				checkout scmGit(
					branches: [[name: '*/master']], 
					extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'automation']], 
					userRemoteConfigs: [[credentialsId: 'amalik-bitbucket', 
					url: 'http://amalik@ca.liquibase.net:7990/scm/ccb/ccb_automation_repo.git']]
				)

				echo "Building ${env.JOB_NAME}..."
			}  // end steps
		} // end stage

		stage('Policy Checks') {
			steps {
				sh '''
					ls -alh 
					ls -alh automation
					cat liquibase.properties

					# Deploy DDL from DSA repo
					cd dsa
					liquibase flow --flow-file=ddl.flowfile.yaml

					# Run Liquibase flow file
 					liquibase flow --flow-file=automation/checks.flowfile.yaml
				'''  
				
			} // end steps
		} // end stage

    }  // end stages
	
	// post {
	// 		cleanup {  
	// 			archiveArtifacts '**/reports/**'
	// 		} // cleanup	
	
	// } // end post
	
} // end pipeline