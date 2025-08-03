pipeline {
    agent any
    tools {
        jfrog 'jfrog'  
        nodejs 'np'
    }
    environment {
        ARTIFACTORY_URL = 'https://hts2.jfrog.io'
        REPO_NAME = 'manu-npm'
        ARTIFACTORY_USERNAME = 'manu' // Correct credentials ID for username
        ARTIFACTORY_PASSWORD = 'Password@123' // Correct credentials ID for password
        PACKAGE_NAME = 'jquery' // Replace with your package name
        TARGET_REPO_URL = 'https://github.com/manuchandra/manu-yarn-old.git' // URL to your specific repo
        TARGET_DIRECTORY = 'manu-yarn-old' // directory where package.json is located
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Remove the existing directory if it exists
            if (fileExists('manu-yarn-old')) {
                sh 'rm -rf manu-yarn-old' // Warning: This deletes the directory!
            }
                    // Clone the specific repository
                    sh "git clone ${TARGET_REPO_URL}"
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                script {
                    // Change to the directory containing package.json
                    dir(TARGET_DIRECTORY) {
                        // Install Yarn globally and then install dependencies
                        sh 'npm install -g npm@latest'
                        sh 'npm install -g yarn@^2.4.0'
                        sh 'yarn --version'
                        sh 'yarn install'
                    }
                }
            }
        }
        stage('Build Package') {
            steps {
                script {
                    // Change to the directory containing package.json
                    dir(TARGET_DIRECTORY) {
                        sh 'yarn build' // Replace with your build command if necessary
                    }
                }
            }
        }
         stage('Install JFrog CLI') {
            steps {
                // Only include this if you didn't setup using tools
                script {
                    sh '''
                    curl -fL https://install-cli.jfrog.io | sh
                    jf --version
                    '''
                }
            }
        }
        stage('Publish to Artifactory') {
            steps {
                script {
                    // Change to the directory containing package.json
                    dir(TARGET_DIRECTORY) {
                        // Configure JFrog CLI
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
