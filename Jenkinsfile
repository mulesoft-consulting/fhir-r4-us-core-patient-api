pipeline {
  agent {
    label 'mule-builder'
  }
  environment {
    DEPLOY_CREDS = credentials('deploy-anypoint-user')
    MULE_VERSION = '4.3.0'
    BG = "1Platform\\1HLS\\Payer"
    SECURE_PROPERTIES_KEY = credentials('fhir-payer-encryption-key')
    PLATFORM_USERNAME = DEPLOY_CREDS_USR
    PLATFORM_PASSWORD = DEPLOY_CREDS_PSW
  }
  stages {
    stage('Prepare') {
       checkout([  
            $class: 'GitSCM', 
            branches: [[name: 'refs/heads/master']], 
            doGenerateSubmoduleConfigurations: false, 
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'fhir-parent-pom']], 
            submoduleCfg: [], 
            userRemoteConfigs: [[url: 'https://github.com/mulesoft-fhir/fhir-parent-pom']]
        ])
        checkout([  
            $class: 'GitSCM', 
            branches: [[name: 'refs/heads/master']], 
            doGenerateSubmoduleConfigurations: false, 
            extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'fhir-resource-crud-operations']], 
            submoduleCfg: [], 
            userRemoteConfigs: [[url: 'https://github.com/mulesoft-fhir/fhir-resource-crud-operations']]
        ])
        withMaven(
          mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
            sh 'cd fhir-parent-pom && mvn install'
          }
        withMaven(
          mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
            sh 'cd fhir-resource-crud-operations && mvn install'
          }
    }
    stage('Build') {
      steps {
        withMaven(
          mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
            sh 'mvn clean -DskipTests package'
          }
      }
    }

    stage('Test') {
      steps {
        withMaven(
          mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
            sh "mvn -B test"
        }
      }
    }

    stage('Deploy Development') {
      when {
        branch 'jenkins-setup'
      }
      environment {
        PLATFORM_ENV = 'Development'
        SETTINGS_ENV = 'dev'
        ANYPOINT_ENV = credentials('DEV_ANYPOINT_SALES')
        APP_NAME = 'sandbox-1hls-fhir-r4-patient-api-v1'
        PLATFORM_CLIENT_ID = credentials('SANDBOX_PLATFORM_CLIENT_ID')
        PLATFORM_CLIENT_SECRET = credentials('SANDBOX_PLATFORM_CLIENT_SECRET')
      }
      steps {
        withMaven(
          mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
            sh 'mvn deploy -e -DmuleDeploy'
          }
      }
    }
    stage('Deploy Production') {
        when {
          branch 'master'
        }
        environment {
          ENVIRONMENT = 'Production'
          ANYPOINT_ENV = credentials('PRD_ANYPOINT_SALES')
          APP_NAME = 'nto-customer-api-v1'
        }
        steps {
          withMaven(
            mavenSettingsConfig: 'f007350a-b1d5-44a8-9757-07c22cd2a360'){
              sh 'mvn -V -B -DskipTests deploy -DmuleDeploy -Dmule.version=$MULE_VERSION -Danypoint.username=$DEPLOY_CREDS_USR -Danypoint.password=$DEPLOY_CREDS_PSW -Dcloudhub.app=$APP_NAME -Dcloudhub.environment=$ENVIRONMENT -Denv.ANYPOINT_CLIENT_ID=$ANYPOINT_ENV_USR -Denv.ANYPOINT_CLIENT_SECRET=$ANYPOINT_ENV_PSW -Dcloudhub.bg=$BG -Dcloudhub.worker=$WORKER -Dapp.client_id=$APP_CLIENT_CREDS_USR -Dapp.client_secret=$APP_CLIENT_CREDS_PSW'
            }
        }
    }
  }

  post {
    always {
      step([$class: 'hudson.plugins.chucknorris.CordellWalkerRecorder'])
    }
  }

  tools {
    maven 'M3'
  }
}