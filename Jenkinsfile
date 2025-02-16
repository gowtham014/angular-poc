pipeline{
  agent any
  parameters {
        choice(name: 'BRANCH_NAME', choices: ['main', 'dev', 'prod'], description: 'Select branch')
  }
  stages{
    stage('checkout'){
      steps{
        git branch:"${params.BRANCH_NAME}",url:'https://github.com/gowtham014/angular-poc.git'
      }
    }
    stage ('sonar-scan') {
      steps{
        withSonarQubeEnv('sonar2') {
          sh """
          npm install
          npm install --save-dev sonar-scanner
          npm run sonar
          """
        }
      }
    }
    stage('Quality Gate') {
      steps {
        script {
            // waitForQualityGate abortPipeline: true
          timeout(time: 5, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Quality Gate failed. Stopping the pipeline!"
            }
          }
        }
      }
    }
    stage ('build') {
      steps{
        sh """
        npm install 
        npm run build
        """
      }
    }
    stage ('packaging'){
      steps {
        sh "cd dist/ && zip -r angular-app.zip ."
      }
    }
    stage('Upload to Nexus') {
      steps {
        nexusArtifactUploader(
          nexusVersion: 'nexus3',
          protocol: 'http',
          nexusUrl: '34.44.201.119:8081',
          groupId: 'DEV',
          version: "${env.BUILD_ID}",
          repository: 'angular-app',
          credentialsId: 'nexus-credentials',
          artifacts: [
            [
              artifactId: 'angular',
              classifier: '',
              file: 'dist/angular-app.zip',
              type: 'zip'
            ]
            
          ]
        )
      }
    }
    }
  post {
    always {
      cleanWs() 
    }
  }
}