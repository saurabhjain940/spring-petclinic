pipeline {
    agent {
        docker { 
            alwaysPull true
            image '730335384723.dkr.ecr.ap-south-1.amazonaws.com/jenkins/agent:latest'
            registryUrl 'https://730335384723.dkr.ecr.ap-south-1.amazonaws.com'
            registryCredentialsId 'ecr:ap-south-1:aws'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u 0:0'
            }
    }

    environment {
        AWS_REGION = 'ap-south-1'                 // AWS Region
        AWS_ACCOUNT_ID = '730335384723'             // AWS account ID
        //SNS_TOPIC_NAME = ''             // Amazon SNS if pipeline success Notification 
        DEPLOY_GITOPS_REPO = "skj-k8s" // application helm charts repo
        DEPLOY_TARGET_FILE = "manifest.yaml"         // Target values file (Ex: microservices/applications/sub-folder/Service/values.yaml)
        DEPLOY_TARGET_BRANCH = "main" // Branch name
        REGISTRY = "730335384723.dkr.ecr.ap-south-1.amazonaws.com"                   // Amazon ECR Registry 
        REPOSITORY = "hybrid"             // Amazon ECR Repo name
        //BUILD_JENKINSFILE_PATH = "jenkinsfile-(microservice-name)" // Path to this specific Jenkinsfile of application needs to build
    }


    stages {

        stage('Checkout Code') {
            steps {
                // Get the latest code from your version control system
                checkout scm
            }
        }
        stage('Install dependencies') {
            steps {
                echo 'Install..'
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
                 sh """
                # Fetching the latest commit ID
                echo 'Fetching the latest commit ID...'
                git config --global --add safe.directory '*'
                commitId=\$(git log -n1 --format='%h')
                date=\$(date -u +'_%Y_%m_%d_%H_%M')
                IMAGE_TAG=v_\${commitId}\${date}
                echo 'Generated IMAGE_TAG: \$IMAGE_TAG'

                # Create a file to store the IMAGE_TAG
                touch ~/IMAGE_TAG.txt && echo \$IMAGE_TAG > ~/IMAGE_TAG.txt

                # Display the generated image and tag
                echo 'Image and tag to be built: \$REGISTRY/\$REPOSITORY:\$IMAGE_TAG'

                # Explicitly specify the build argument
                echo 'Building Docker image...'
                #docker build -t \$REGISTRY/\$REPOSITORY:\$IMAGE_TAG .
                docker tag 730335384723.dkr.ecr.ap-south-1.amazonaws.com/hybrid:v_0d29b6a_2025_03_13_19_51 \$REGISTRY/\$REPOSITORY:\$IMAGE_TAG
            """
            }
        }
        stage('Push') {
            steps {
                echo 'Building..'
                sh '''
                IMAGE_TAG=`cat ~/IMAGE_TAG.txt` 
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 730335384723.dkr.ecr.ap-south-1.amazonaws.com
                docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
                '''
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
                withCredentials([file(credentialsId: 'ssh-key', variable: 'key')]) {
                    sh '''
                        # Create a directory for SSH keys and copy the key to it, setting proper permissions
                        mkdir ~/.ssh/ && cat ${key} > ~/.ssh/id_rsa && chmod 400 ~/.ssh/id_rsa
                        echo "SSH key copied and permissions set."

                        # Display the current workspace directory
                        echo "Current workspace directory: $WORKSPACE"

                        # Display the current user
                        whoami

                        # Cleanup if DEPLOY_GITOPS_REPO already exists
                        # Check if the directory exists
                        if [ -d $DEPLOY_GITOPS_REPO ]; then
                            echo "Directory exists. Deleting..."
                            # Add any commands to delete the directory here
                            rm -r $DEPLOY_GITOPS_REPO
                        else
                            echo "Directory does not exist. Proceeding..."
                            # Add your code to proceed with other tasks here
                        fi

                        # Clone the GitOps repository using the configured SSH key
                        GIT_SSH_COMMAND="ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no" git clone git@github.com:saurabhjain940/$DEPLOY_GITOPS_REPO.git
                        echo "GitOps repository cloned."

                        # Change directory to the GitOps repository
                        cd $DEPLOY_GITOPS_REPO

                        # Configure Git globally to add a safe directory and set user email and name
                        git config --global --add safe.directory '*' 
                        git config --global user.email "saurabhjain940@gmail.com"
                        git config --global user.name "saurabhjain940"
                        # Read the image tag from the file
                        IMAGE_TAG=`cat ~/IMAGE_TAG.txt`  
                        echo "Setting \"$IMAGE_TAG\" tag in \"$DEPLOY_TARGET_FILE\""

                        # Replace the "tag" value in the specified file with the new image tag
                        sed -i 's|image: 730335384723.dkr.ecr.ap-south-1.amazonaws.com/hybrid:.*|image: 730335384723.dkr.ecr.ap-south-1.amazonaws.com/hybrid:'"$IMAGE_TAG"'|' $DEPLOY_TARGET_FILE



                        # Display the updated file contents with the new tag
                        cat $DEPLOY_TARGET_FILE

                        # Check if there are any changes to commit
                        if git diff --quiet; then
                            echo "No changes to commit"
                        else
                            # Display Git status
                            git status

                            # Stage the updated file for commit
                            git add $DEPLOY_TARGET_FILE

                            # Commit the changes with a descriptive message
                            git commit -m "Update Tag value in values.yml"

                            # Push the changes to the GitOps repository
                            git push 
                            echo "Changes pushed to GitOps repository."
                        fi
                    '''

                }
            }
        }
    }
}
