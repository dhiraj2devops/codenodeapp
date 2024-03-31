# codenodeapp

Agent: The pipeline can run on any available agent.

Triggers: The pipeline is triggered by a GitHub push event using githubPush().

Tools: The pipeline uses Node.js tools configured for the 'codenodeapp' project.

Stages:

Checkout: This stage checks out the code from the specified Git repository (https://github.com/dhiraj2devops/codenodeapp.git) and branch (*/master).

Build: In this stage, the command npm install is executed to install project dependencies.

Test: The pipeline runs tests using Mocha (./node_modules/mocha/bin/_mocha --exit ./test/test.js).

Deploy: This stage involves deployment tasks using SSH commands:
Connects to a remote server (ec2-user@54.175.175.71) with SSH options (-o StrictHostKeyChecking=no to avoid strict host key checking).
Changes directory to /home/ec2-user/codenodeapp on the remote server.
Performs a Git pull to fetch the latest code from the GitHub repository.
Installs PM2 globally (sudo npm install -g pm2).
Restarts the application using PM2 (pm2 restart index.js).
******************************************************************************************

use for following pipeline code to deploy the nodejs application

pipeline {
    agent any

triggers {
    githubPush()
}
 
tools{
    nodejs 'codenodeapp'
}
    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/dhiraj2devops/codenodeapp.git']])
            }
        }
        stage('build') {
            steps {
                sh 'npm install'
            }
        }
        stage('test') {
            steps {
                sh './node_modules/mocha/bin/_mocha --exit ./test/test.js'
            }
        }
        stage('deploy') {
            steps {
                script {
                    sshagent(['a3d73337-1106-44c7-af5d-d23b0ce88c6e']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@54.175.175.71  << EOF
                        cd /home/ec2-user/codenodeapp 
                        git pull https://github.com/dhiraj2devops/codenodeapp.git
                        sudo npm install -g pm2 
                        pm2 restart index.js
                        << EOF
                    '''                    
                }
                
            }
        }
    }
}
}
