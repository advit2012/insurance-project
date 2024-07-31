node{
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    stage('Prepare environment')
       echo "initialise all variable"
        mavenHome = tool name: 'maven' , type: 'maven'
        mavenCMD ="${mavenHome}/bin/mvn"
        docker = tool name: 'docker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName="1.0"
        
    stage('Code Checkout'){
        echo "checkout from git repo"
        git 'https://github.com/advit2012/insurance-project.git'
        }
      stage('Build the Application'){
        echo "Cleaning... Compiling...Testing... Packaging..."
        //sh 'mvn clean package'
        sh "${mavenCMD} clean package"     
    }
      stage('publish the report'){
          echo "generating test reports"
          publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/insureme project/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
      }
      stage('Containerise the application'){
          echo "making the image out of the application"
          sh "${dockerCMD} build -t advit2012/insureme:${tagName} . "
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

      stage('Configure and Deploy to the test-serverusing ansible'){  
          ansiblePlaybook become: true, credentialsId: 'ansible-key', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
      }

}
}


