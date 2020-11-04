pipeline {
  agent {
    label 'mule-builder'
  }
  environment {
    DEPLOY_CREDS = credentials('deploy-anypoint-user')
    MULE_VERSION = '4.3.0'
    BG = "1Platform\\1HLS\\Payer"
    SECURE_PROPERTIES_KEY = credentials('fhir-payer-encryption-key')
    PLATFORM_USERNAME = '${DEPLOY_CREDS_USR}'
    PLATFORM_PASSWORD = '${DEPLOY_CREDS_PSW}'
  }
  stages {
    stage('Prepare') {
       steps {
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
          mavenSettingsConfig: 'certified-mvn-settings.xml'){
            sh 'cd fhir-parent-pom && mvn install'
          }
        withMaven(
          mavenSettingsConfig: 'certified-mvn-settings.xml'){
            sh 'cd fhir-resource-crud-operations && mvn install'
          }
       }
    }
    stage('Build') {
      steps {
        withMaven(
          mavenSettingsConfig: 'certified-mvn-settings.xml'){
            sh 'mvn -B clean -DskipTests package'
          }
      }
    }

    stage('Test') {
      steps {
        withMaven(
          mavenSettingsConfig: 'certified-mvn-settings.xml'){
            sh "mvn -B test"
        }
      }
    }

    stage('Deploy Development') {
      when {
        branch 'dev'
      }
      environment {
        PLATFORM_ENV = 'Development'
        SETTINGS_ENV = 'dev'
        APP_NAME = 'dev-1hls-fhir-r4-patient-api-v1'
        PLATFORM_CLIENT_ID = credentials('SANDBOX_PLATFORM_CLIENT_ID')
        PLATFORM_CLIENT_SECRET = credentials('SANDBOX_PLATFORM_CLIENT_SECRET')
      }
      steps {
        withMaven(
          mavenSettingsConfig: 'certified-mvn-settings.xml'){
            sh 'mvn -B deploy -e -DmuleDeploy -DskipTests'
          }
      }
    }
    stage('Deploy Production') {
      when {
        branch 'master'
      }
      environment {
        PLATFORM_ENV = 'Production'
        SETTINGS_ENV = 'prod'
        APP_NAME = '1hls-fhir-r4-patient-api-v1'
        PLATFORM_CLIENT_ID = credentials('PRODUCTION_PLATFORM_CLIENT_ID')
        PLATFORM_CLIENT_SECRET = credentials('PRODUCTION_PLATFORM_CLIENT_SECRET')
      }
      steps {
        withMaven(
          mavenSettingsConfig: 'certified-mvn-settings.xml'){
            sh 'mvn -B deploy -e -DmuleDeploy -DskipTests'
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