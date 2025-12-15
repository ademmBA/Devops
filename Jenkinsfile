pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKER_HUB_CREDENTIALS = 'your-credintials' // ID des credentials Jenkins pour Docker Hub
        DOCKER_IMAGE_NAME = 'your-name/student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code depuis GitHub...'
                checkout scm
            }
        }

        stage('Build sans tests') {
            steps {
                echo 'Compilation avec Maven...'
                sh 'mvn -B clean install -DskipTests'
            }
        }

        stage('Build & SonarQube analysis') {
            steps {
                echo 'Analyse SonarQube en cours...'
                withSonarQubeEnv('My SonarQube Server') {
                    sh '''
                        mvn -B clean verify sonar:sonar \
                            -Dspring.datasource.url="jdbc:mysql://my-mysql:3306/studentdb?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC" \
                            -Dspring.datasource.username=root \
                            -Dspring.datasource.password= \
                            -Dspring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
                    '''
                }
            }
        }

        stage('Archive JAR') {
            steps {
                echo 'Archivage du fichier JAR...'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: false
            }
        }

        stage('Docker Build & Push') {
            steps {
                echo 'Construction et push de l’image Docker...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        def customImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def qgStatus = 'NON DÉTECTÉ'
                try {
                    def qg = waitForQualityGate(abortPipeline: false)
                    qgStatus = qg.status
                } catch (err) {
                    echo "Impossible de récupérer le Quality Gate : ${err}"
                }
            }
        }

        success {
            echo 'BUILD RÉUSSI – Tout est vert !'
        }

        failure {
            echo 'Le build a échoué'
        }
    }
}
