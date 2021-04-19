pipeline {
    agent { label 'slave01'}
    environment {
    PROJECT_NAME = 'project'
	DOCKER_REPO = 'demodocker'
	MS_NAME = 'back'
	DOCKER_REPO_PASSWORD = credentials('docker_api_token') 
    KUBETEST_CREDS = credentials('kubetest_config')
    FULL_PATH_BRANCH = "${sh(script:'git name-rev --name-only HEAD', returnStdout: true)}" 
    GIT_BRANCH = FULL_PATH_BRANCH.substring(FULL_PATH_BRANCH.lastIndexOf('/') + 1, FULL_PATH_BRANCH.length())
    }
    tools {
        maven 'mvn'
        jdk 'jdk11'
    }
  
    stages {

        stage('Setup Jenkins job name') {
            steps {
                script {
                    currentBuild.displayName = "${GIT_BRANCH}-${BUILD_NUMBER}"
                    COMMIT_HASH = sh(script: 'git show -s --format=%H', returnStdout: true).trim()}
                echo "${FULL_PATH_BRANCH}"
                echo "${COMMIT_HASH}"
                echo "$env.GIT_BRANCH" }}


        stage('Check Branch Name Develop') {
         when { expression { env.GIT_BRANCH =~  /(develop)/ }}
            steps {
                script { DEPLOY_ENV = 'dev' }
                echo "${DEPLOY_ENV}"
            }}

        stage ('Build') {
            steps {
                sh """
                    mvn --settings mvnsettings.xml clean  package -P ${DEPLOY_ENV}
                    #mvn verify
                   """
            }
        }
       stage('Sonarqube') {
            when {
               expression { env.GIT_BRANCH =~  /(develop)/ }
            }
            environment {
              scannerHome = tool 'SonarQube Scanner'
             }
           steps {
                sh '''
                   ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=$PROJECT_NAME -Dsonar.projectName=$PROJECT_NAME -Dproject.settings=./sonar-project-local.properties
                  '''
            }
      }
     }
    post {
        always {
            cleanWs()
        }
    }
     
     }
