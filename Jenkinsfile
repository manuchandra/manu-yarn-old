pipeline {
    agent any
    tools {
        jfrog 'jfrog-cli-2.78.1'
        nodejs 'np'
    }
    environment {
        ARTIFACTORY_URL = 'https://hts2.jfrog.io'
        REPO_NAME = 'manu-yarn-npm'
        ARTIFACTORY_USERNAME = 'manu'
        ARTIFACTORY_PASSWORD = 'Password@123'
        PACKAGE_NAME = 'jquery'
    }
    stages {
        stage('Setup Environment and Dependencies') {
            steps {
                dir('/var/jenkins_home/workspace/manu-yarn-old') {
                    sh '''
                    rm -rf node_modules yarn.lock
                    npm install -g npm@latest
                    npm install -g yarn@^2.4.0
                    yarn cache clean
                    yarn install
                    '''
                }
            }
        }
        stage('Build Package') {
            steps {
                dir('/var/jenkins_home/workspace/manu-yarn-old') {
                    sh 'yarn build' // Ensure this command is correct for your application.
                }
            }
        }
        stage('Install JFrog CLI') {
            steps {
                sh '''
                curl -fL https://install-cli.jfrog.io | sh
                jf --version
                '''
            }
        }
        stage('Publish to Artifactory') {
            steps {
                dir('/var/jenkins_home/workspace/manu-yarn-old') {
                    sh """
                    jf c add --insecure-tls true --url $ARTIFACTORY_URL --user $ARTIFACTORY_USERNAME --password $ARTIFACTORY_PASSWORD --interactive=false
                    jf yarn-config --repo-resolve $REPO_NAME
                    jf yarn install --build-name=my-build --build-number=1
                    jf rt bce my-build 1
                    jf rt bp my-build 1
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Build and publish to Artifactory was successful!'
        }
        failure {
            echo 'Build or publish to Artifactory failed.'
        }
    }
}
