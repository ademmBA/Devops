pipeline {
    agent any

    stages {
        stage('Scan') {
            steps {
                withSonaQube(installationNama: 'sql'} {
                    sh './nvnw clean org.sonarsource.scanne.maven:sonar-maven-plugin:3.9.0.2155:sonar'
                }
            }
        }
    }
}
//////////////
