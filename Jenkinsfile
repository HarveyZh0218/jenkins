//镜像版本号
def tag = "latest"

node {
    stage('Clone Code') {
        echo "1.Clone Stage"
        //克隆 代码
        //credentialsId: auth 
        git branch: 'master', changelog: false, credentialsId: 'github', poll: false, url: 'https://github.com/HarveyZh0218/jenkins.git'
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build Gradle') {
        echo "Build Gradle"
        sh "chmod +x gradlew"
        sh "./gradlew clean build"
    }
    stage('Build Image') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t zhanghu0218/ktor-jenkins:${build_tag} ."
    }
    //push docker镜像到DockerHub
    stage('Push Docker Images to DockerHub') {
        echo "4.Push Docker Image Stage"
        //定义镜像名
        //def ImageNmae = "${project_name}:${tag}"
      
        withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
            sh "docker push zhanghu0218/ktor-jenkins:${build_tag}"
        }
    }
    //push docker镜像到Nexus
    // stage('Push Docker Images to Nexus Registry'){
    //     sh 'docker login -u user -p password http://localhost:8081/repository/docker-private/'
    //     sh 'docker push http://localhost:8081/repository/docker-private//ktor-jenkins:${build_tag}}'
    //     sh 'docker rmi $(docker images --filter=reference="http://localhost:8081/repository/docker-private//ktor-jenkins:${build_tag}*" -q)'
    //     sh 'docker logout http://localhost:8081/repository/docker-private/'
    // }
    stage('Deploy') {
        echo "5. Deploy Stage"
        // def userInput = input(
        //     id: 'userInput',
        //     message: 'Choose a deploy environment',
        //     parameters: [
        //         [
        //             $class: 'ChoiceParameterDefinition',
        //             choices: "Dev\nQA\nProd",
        //             name: 'Env'
        //         ]
        //     ]
        // )
        // echo "This is a deploy step to ${userInput}"
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        // if (userInput == "Dev") {
        //     // deploy dev stuff
        // } else if (userInput == "QA"){
        //     // deploy qa stuff
        // } else {
        //     // deploy prod stuff
        // }
        sh "kubectl apply -f k8s.yaml"
    }
}
