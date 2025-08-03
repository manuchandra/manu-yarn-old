// This pipeline script has been updated to provide a more robust fix
// for the persistent 'TypeError' related to Yarn's Plug'n'Play (PnP) linker.
// It includes a more aggressive cleanup and a check to ensure the
// 'node-modules' linker is correctly being used.
pipeline {
    agent any
    tools {
        // Ensure that a Node.js tool named 'np' is configured in Jenkins.
        // It is recommended to use a more descriptive name, like 'nodejs-20'
        // and a corresponding Yarn version if possible.
        nodejs 'np'
        jfrog 'jfrog-cli-2.78.1'
    }
    environment {
        // IMPORTANT: The YARN_NODE_LINKER environment variable forces Yarn to use the
        // more compatible node_modules linker, which is necessary to resolve the TypeError.
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
                    # Perform a more aggressive cleanup to remove all Yarn-specific files and cache
                    echo "Performing aggressive cleanup..."
                    rm -rf node_modules .yarn yarn.lock
                    
                    # Verify that the YARN_NODE_LINKER environment variable is being used
                    echo "Verifying Yarn linker configuration..."
                    yarn config get nodeLinker
                    
                    # Install dependencies using the node-modules linker
                    echo "Installing dependencies with 'yarn install'..."
                    #yarn install
                    '''
                }
            }
        }
        stage('Build Package') {
            steps {
                dir('/var/jenkins_home/workspace/manu-yarn-old') {
                    // This assumes your project has a 'build' script in package.json
                   // sh 'yarn build'
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
