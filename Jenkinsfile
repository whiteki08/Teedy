def mvnCommand(String args) {
  return """
    set -eu
    MAVEN_VERSION=3.9.11
    MAVEN_HOME=\"\$PWD/.maven/apache-maven-\${MAVEN_VERSION}\"
    if [ ! -x \"\$MAVEN_HOME/bin/mvn\" ]; then
      mkdir -p \"\$PWD/.maven\"
      curl -fsSL \"https://archive.apache.org/dist/maven/maven-3/\${MAVEN_VERSION}/binaries/apache-maven-\${MAVEN_VERSION}-bin.tar.gz\" | tar -xz -C \"\$PWD/.maven\"
    fi
    export PATH=\"\$MAVEN_HOME/bin:\$PATH\"
    mvn ${args}
  """.stripIndent()
}

pipeline {
  agent any
  stages {
    stage('Clean') {
      steps {
        sh mvnCommand('clean')
      }
    }
    stage('Compile') {
      steps {
        sh mvnCommand('compile')
      }
    }
    stage('Test') {
      steps {
        sh mvnCommand('test -Dmaven.test.failure.ignore=true')
      }
    }
    stage('Package') {
      steps {
        sh mvnCommand('package -DskipTests')
      }
    }
    stage('Site') {
      steps {
        // site: generates all reports (surefire, jacoco, javadoc, pmd)
        // post-site: runs antrun to create jacoco/index.html stub
        // site:stage: copies everything into target/staging
        sh mvnCommand('site post-site site:stage -Dmaven.test.failure.ignore=true')
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'target/staging/**/*.*', fingerprint: true, allowEmptyArchive: true
      archiveArtifacts artifacts: '**/target/**/*.jar', fingerprint: true, allowEmptyArchive: true
      archiveArtifacts artifacts: '**/target/**/*.war', fingerprint: true, allowEmptyArchive: true
      script {
        def hasTestReports = sh(script: "find . -path '*/target/surefire-reports/*.xml' -size +0c | grep -q .", returnStatus: true) == 0
        if (hasTestReports) {
          junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
        } else {
          echo 'WARNING: No JUnit test reports found in surefire-reports.'
        }
      }
    }
  }
}