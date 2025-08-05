pipeline {
    agent any
    tools {
        jfrog 'jfrog-cli-2.78.1'  
        nodejs 'np'
    }
    environment {
        ARTIFACTORY_URL = 'https://hts2.jfrog.io'
        REPO_NAME = 'manu-yarn-npm'
        ARTIFACTORY_USERNAME = 'manu' // Correct credentials ID for username
        ARTIFACTORY_PASSWORD = 'Password@123' // Correct credentials ID for password
    }
    stages {
        stage('Setup Environment and Dependencies') {
            steps {
                script {
                    // Change to the directory containing package.json
                    dir('/var/jenkins_home/workspace/manu-yarn-old') {
                        // Install Yarn globally and then install dependencies
                       // sh 'npm install -g npm@latest'
                        sh 'npm cache clean --force'
                        sh 'corepack enable'
                        sh 'corepack prepare yarn@4.9.2 --activate'
                        //sh 'npm install -g yarn@4.9.2'
                        sh 'yarn cache clean' 
                        sh 'yarn --version'
                        sh 'yarn install --immutable'
                    }
                }
            }
        }
        stage('Build Package') {
            steps {
                script {
                    // Change to the directory containing package.json
                    dir('/var/jenkins_home/workspace/manu-yarn-old') {
                       sh 'yarn run' // Replace with your build command if necessary
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
                    dir('/var/jenkins_home/workspace/manu-yarn-old') {
                        // Configure JFrog CLI
                        sh """
                        #jf yarn add --dev lerna
                        #jf yarn lerna version -y --conventional-commits
                        jf c add --insecure-tls true --url $ARTIFACTORY_URL --user $ARTIFACTORY_USERNAME --password $ARTIFACTORY_PASSWORD --interactive=false
                        jf yarn-config --repo-resolve $REPO_NAME
                        JFROG_CLI_LOG_LEVEL=DEBUG  jf yarn install --build-name=my-build --build-number=1 
                        jf rt bce ${JOB_NAME} ${BUILD_NUMBER}
                        dd if=/dev/zero of=package.tgz bs=1M count=1
                        dd if=/dev/zero of=file.tgz bs=1M count=1
                        jf rt u "*.tgz" "$REPO_NAME/${JOB_NAME}/" --build-name=${JOB_NAME} --build-number=${BUILD_NUMBER}
                        jf rt bp ${JOB_NAME} ${BUILD_NUMBER}
                        jf rt build-discard --max-builds=1 ${JOB_NAME} --delete-artifacts=true
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
