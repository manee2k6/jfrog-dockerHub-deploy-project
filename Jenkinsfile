node {
    def commit_id
    def server = Artifactory.server('windevops.jfrog.io')
    def rtGradle = Artifactory.newGradleBuild()
    def buildInfo = Artifactory.newBuildInfo()
    stage 'Build'
        git url: 'https://github.com/wardviaene/gs-gradle.git'

    stage 'Artifactory configuration'
        rtGradle.tool = 'gradle' // Tool name from Jenkins configuration
        rtGradle.deployer repo:'gradle-dev-local',  server: server
        rtGradle.resolver repo:'gradle-dev', server: server

        stage('Config Build Info') {
            buildInfo.env.capture = true
            buildInfo.env.filter.addInclude("*")
        }

        stage('Extra gradle configurations') {
            rtGradle.usesPlugin = true // Artifactory plugin already defined in build script
        }
        stage('Exec Gradle') {
            rtGradle.run rootDir: "artifactory/", buildFile: 'build.gradle', tasks: 'clean artifactoryPublish', buildInfo: buildInfo
        }
        stage('Publish build info') {
            server.publishBuildInfo buildInfo
        }
    stage ('Docker Preparation'){
        git url: 'https://github.com/devops-courses/nodejs-docker-demo.git'
        sh "git rev-parse --short HEAD > .git/commit-id"                        
        commit_id = readFile('.git/commit-id').trim()
    }
    stage('Build Docker Image') {
     nodejs(nodeJSInstallationName: 'nodejs') {
       sh 'npm install --only=dev'
       sh 'npm test'
     }
    }
    stage('Test execution'){
         nodejs(nodeJSInstallationName: 'nodejs'){  
            sh 'npm test'
      }
    }
     stage('Docker build/push') {
       docker.withRegistry('https://index.docker.io/v1/', 'DockerID') {
       def app = docker.build("manee2k6/docker-nodejs:app-${commit_id}", '.').push()
     }
   }
    
}
