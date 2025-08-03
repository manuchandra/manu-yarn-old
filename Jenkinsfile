// This pipeline resolves the 'TypeError: (0 , o.isDate) is not a function' error
// by forcing Yarn to use the node_modules linker instead of the default Plug'n'Play (PnP) linker.
pipeline {
    agent any
    tools {
        // Ensure that a Node.js tool named 'np' is configured in Jenkins.
        // It's recommended to use a more descriptive name, like 'nodejs-20'
        // and a corresponding Yarn version if possible.
        nodejs 'np'
        jfrog 'jfrog-cli-2.78.1'
    }
    environment {
        // IMPORTANT: The YARN_NODE_LINKER environment variable forces Yarn to use the
        // more compatible node_modules linker, which resolves the TypeError.
        YARN_NODE_LINKER = 'node-modules'
        
        // JFrog Artifactory Configuration
        ARTIFACTORY_URL = 'https://hts2.jfrog.io'
        REPO_NAME = 'manu-yarn-npm'
        
        // It is highly recommended to use Jenkins credentials for sensitive information
        // instead of hardcoding them in the pipeline script.
        // For example: withCredentials([usernamePassword(credentialsId: 'jfrog-creds', passwordVariable: 'ARTIFACTORY_PASSWORD', usernameVariable: 'ARTIFACTORY_USERNAME')])
        ARTIFACTORY_USERNAME = 'manu'
        ARTIFACTORY_PASSWORD = 'Password@123'
    }
    stages {
        stage('Setup Dependencies') {
            steps {
                // We use 'sh' inside 'dir' to ensure commands run in the correct workspace.
                dir('/var/jenkins_home/workspace/manu-yarn-old') {
                    sh '''
                    # Clean up existing dependencies and lock files
                    rm -rf node_modules .yarn/cache yarn.lock
                    # The `npm install` for yarn is not strictly necessary if 'np' tool provides it.
                    # We will rely on the configured 'np' tool for the node environment.
                    # The 'yarn' command from the tool should be available.
                    
                    # Install dependencies using the node-modules linker
                    echo "Installing dependencies with 'yarn install' using node-modules linker..."
                    yarn install
                    '''
                }
            }
        }
        stage('Build Package') {
            steps {
                dir('/var/jenkins_home/workspace/manu-yarn-old') {
                    // This assumes your project has a 'build' script in package.json
                    sh 'yarn build'
                }
            }
        }
        stage('Publish to Artifactory') {
            steps {
                dir('/var/jenkins_home/workspace/manu-yarn-old') {
                    sh """
                    # Configure JFrog CLI with Artifactory details
                    echo "Configuring JFrog CLI..."
                    jf c add my-jfrog-server --url=$ARTIFACTORY_URL --user=$ARTIFACTORY_USERNAME --password=$ARTIFACTORY_PASSWORD --interactive=false --insecure-tls=true
                    
                    # Publish the package using the configured Artifactory repository
                    echo "Publishing to Artifactory..."
                    jf rt yarn publish --repo=$REPO_NAME
                    
                    // You can optionally continue with build info steps if needed
                    // jf rt bce my-build 1
                    // jf rt bp my-build 1
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Build and publish to Artifactory was successful! 🎉'
        }
        failure {
            echo 'Build or publish to Artifactory failed. 😞'
        }
    }
}
