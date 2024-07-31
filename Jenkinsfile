node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName

    stage('Prepare environment') {
        echo "initialise all variables"
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "1.0"
    }

    stage('Code Checkout') {
        echo "checkout from git repo"
        git 'https://github.com/advit2012/insurance-project.git'
    }

    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish the Report') {
        echo "generating test reports"
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }

    stage('Containerise the Application') {
        echo "making the image out of the application"
        sh "${dockerCMD} build -t advit2012/insureme:${tagName} ."
    }

   stage('Pushing it to DockerHub') {
    echo 'Pushing the docker image to DockerHub'
    withCredentials([string(credentialsId: 'dockerhubpassword', variable: 'dockerhubpassword')]) {
        // Login to DockerHub
        sh '''
            echo "${dockerhubpassword}" | ${dockerCMD} login -u advit2012 --password-stdin
        '''
        // Push Docker image
        sh "${dockerCMD} push advit2012/insureme:${tagName}"
    
        }
    }

    stage('Configure and Deploy to the Test Server using Ansible') {
        ansiblePlaybook(
            become: true,
            credentialsId: 'ansible-key',
            disableHostKeyChecking: true,
            installation: 'ansible',
            inventory: '/etc/ansible/hosts',
            playbook: 'ansible-playbook.yml'
        )
    }
}


