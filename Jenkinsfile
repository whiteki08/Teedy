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
    stage('Install') {
      steps {
        // Install all modules to the local repo first so subsequent plugin-only invocations
        // (pmd, jacoco, javadoc, site) can resolve reactor snapshot artifacts locally.
        sh mvnCommand('-B -ntp -U install -DskipTests')
      }
    }
    stage('PMD') {
      steps {
        sh mvnCommand('pmd:pmd')
      }
    }
    stage('JaCoCo') {
      steps {
        sh mvnCommand('jacoco:report')
      }
    }
    stage('Javadoc') {
      steps {
        sh mvnCommand('javadoc:javadoc')
      }
    }
    stage('Site') {
      steps {
        // install first so multi-module snapshot artifacts are available for site generation
        sh mvnCommand('install -DskipTests site')
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