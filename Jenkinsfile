// This updated pipeline script incorporates a more secure approach
// to managing credentials and ensures the build and publish steps
// are correctly enabled.
pipeline {
    agent any
    tools {
        // Ensure that a Node.js tool named 'np' is configured in Jenkins.
        // It's recommended to use a more descriptive name, like 'nodejs-20'.
        nodejs 'np'
        jfrog 'jfrog-cli-2.78.1'
    }
    environment {
        // IMPORTANT: This forces Yarn to use the node_modules linker,
        // which resolves the PnP 'TypeError' issue.
        YARN_NODE_LINKER = 'node-modules'
        
        // JFrog Artifactory Configuration (non-sensitive)
        ARTIFACTORY_URL = 'https://hts2.jfrog.io'
        REPO_NAME = 'manu-yarn-npm'
    }
    stages {
        // We use a withCredentials block to securely handle sensitive
        // information like the username and password, instead of
        // hardcoding them in the pipeline script.
        // NOTE: You must create a 'Username with password' credential
        // in Jenkins with the ID 'jfrog-creds'.
        stage('Build and Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jfrog-creds', passwordVariable: 'ARTIFACTORY_PASSWORD', usernameVariable: 'ARTIFACTORY_USERNAME')]) {
                    dir('/var/jenkins_home/workspace/manu-yarn-old') {
                        // --- Stage 1: Setup Dependencies ---
                        sh '''
                        # Perform an aggressive cleanup to remove any old Yarn-specific files
                        echo "Performing aggressive cleanup..."
                        rm -rf node_modules .yarn yarn.lock
                        
                        # Verify that the YARN_NODE_LINKER environment variable is being used
                        echo "Verifying Yarn linker configuration..."
                        yarn config get nodeLinker
                        
                        # Install dependencies using the node-modules linker
                        echo "Installing dependencies with 'yarn install'..."
                        yarn install
                        '''
                        
                        // --- Stage 2: Build Package ---
                        // This step assumes your project has a 'build' script in package.json.
                        sh 'yarn build'
                        
                        // --- Stage 3: Publish to Artifactory ---
                        sh """
                        # Configure JFrog CLI with Artifactory details
                        echo "Configuring JFrog CLI..."
                        jf c rm my-jfrog-server
                        jf c add my-jfrog-server --url=$ARTIFACTORY_URL --user=$ARTIFACTORY_USERNAME --password=$ARTIFACTORY_PASSWORD --interactive=false --insecure-tls=true
                        
                        # Publish the package using the configured Artifactory repository
                        echo "Publishing to Artifactory..."
                        jf yarn publish --repo=$REPO_NAME
                        
                        // Capture and publish build information for traceability
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
            echo 'Build and publish to Artifactory was successful! 🎉'
        }
        failure {
            echo 'Build or publish to Artifactory failed. 😞'
        }
    }
}
