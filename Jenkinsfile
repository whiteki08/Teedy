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
    stage('Install & Reports') {
      steps {
        // Run a single reactor invocation that installs modules and runs reporting plugins
        // in the same Maven lifecycle so reactor resolution is consistent and no remote
        // snapshot lookups are required.
        sh mvnCommand('-B -ntp -U install -DskipTests pmd:pmd jacoco:report javadoc:javadoc site')
      }
    }
    stage('Package') {
      steps {
        sh mvnCommand('package -DskipTests')
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: '**/target/site/**/*.*', fingerprint: true, allowEmptyArchive: true
      archiveArtifacts artifacts: '**/target/**/*.jar', fingerprint: true, allowEmptyArchive: true
      archiveArtifacts artifacts: '**/target/**/*.war', fingerprint: true, allowEmptyArchive: true
      junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
    }
  }
}