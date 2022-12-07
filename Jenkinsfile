pipeline {
  tools {
    jdk 'openjdk-11'
  }
  agent {
    kubernetes {
      yaml '''
      apiVersion: v1
      kind: Pod
      spec:
        containers:
        - name: mono
          image: mono:latest
          command:
          - sleep
          args:
          - infinity'''
      defaultContainer 'mono'
    }
  }
  stages {
    stage('SCM') {
      steps {
        checkout scm
      }
    }
    stage('SonarQube analysis') {
      steps {
        script {
          def sqScannerMsBuildHome = tool 'sonar_msbuild_4.6';
            withSonarQubeEnv('sonarqube') {
              sh "mono ${sqScannerMsBuildHome}/SonarScanner.MSBuild.exe begin /k:shadowsocks-windows"
              sh "msbuild /t:Rebuild"
              sh "mono ${sqScannerMsBuildHome}/SonarScanner.MSBuild.exe end"
            }
        }
      }
    }
    stage("Quality Gate") {
      steps {
        script {
          timeout(time: 1, unit: 'HOURS') { 
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
              error "Pipeline aborted due to quality gate failure: ${qg.status}"
            } else {
              println("quality gate passed!")
            }
          }
        }
      }
    }
  }
}
