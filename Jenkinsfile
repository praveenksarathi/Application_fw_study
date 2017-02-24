// This is a sample file for Multi-Platform integration Pipeline.
    def DockerRegistryTemp = 'ec2-13-55-19-58.ap-southeast-2.compute.amazonaws.com'
    def DockerRegistryPerm = 'ec2-13-54-206-33.ap-southeast-2.compute.amazonaws.com'

node {
        stage ('Code Pickup') {
        echo 'The Picked Up Code Type is : ${CodeLocType}'
        echo 'The Picked Up Code Path is : ${CodeLocPath}'
        
        if ("${CodeLocType}".toUpperCase()=='GIT') {
         CodeLocPath = CodeLocPath.substring(0, CodeLocPath.indexOf("//")+2) + scmUsername + ":" + scmPassword + "@" +CodeLocPath.substring(CodeLocPath.indexOf("//")+2, CodeLocPath.length());
            try {
                sh 'ls -a | xargs rm -fr'
            }catch (error) {
                sh "git clone ${CodeLocPath} ."
            }
        }else if ("${CodeLocType}".toUpperCase()=='SVN') {
         sh "svn co --username ${scmUsername} --password ${scmPassword} ${CodeLocPath} ."
        }else{
         error 'This Code Type is not supported in our current Implementation , Please Check after some time'
         echo  'Thanks for Understanding'
             }
      }
        
def AppModulePresent = fileExists 'app'
def TestModulePresent = fileExists 'test'
def appPath = ''
def testPath = ''

if (AppModulePresent){
        echo 'App Module is found , assumed that application is present in /app directory'
        appPath = 'app/'
}else {
        echo 'There is no defined Application path , hence it is assumed that application is in current directory'
        appPath = ''
}
if (TestModulePresent){
        echo 'Test Module is found , assumed that Test Cases are Present for the concerned Modules and has to be performed'
        testPath = 'test/'
}else{
        echo 'No Test Modules found , hence it is assumed that no test environment and / or test cases to be performed'
        testPath = ''
}
def buildAndPackagingReq = true
def buildDockerLocation = appPath + 'Dockerfile.build'
def distDockerLocation = appPath + 'Dockerfile.dist'
def singleDockerFile = appPath + 'Dockerfile'

if (fileExists(buildDockerLocation) && fileExists(distDockerLocation)) {
        echo 'Looks like this application contains seperate Build and Distribution Docker Files , its assumed to be developed in compiler dependant programming language.'
        buildAndPackagingReq = true;
} else if (fileExists(singleDockerFile)) {
        echo 'This application contains a single docker file , it is assumed to be developed in compiler in-dependant / interpreter based programming language.'
        buildAndPackagingReq = false;
        distDockerFile = singleDockerFile
} else {
        error 'No Docker File found in the specified path :' + appPath
        buildAndPackagingReq = false;
}

// Copying App Directory to current working directory

def appWorkingDir = (appPath == '') ? '.' : appPath.substring(0,appPath.length()-1)

// NEXUS file for Time Stamp comparison. This file is used for comparing time stamps and differentiating input files from generated output files.
  sh echo 'Nexus>nexus.txt'
  
//End of Initializing
//___________________________________________________________________________________________________________________________________________________________________

if (buildAndPackagingReq) {
        echo 'Compile and Packaging runs as separate entities , it is inferred that the App package is compiler dependant'
        stage ('Build , Test and Package ') {
                echo 'Working Directory for Docker Build file : ' + appWorkingDir
                echo "Build Tag Name : ${dockerRepo}/${dockerImageName}-build:${env.BUILD_NUMBER} "
                echo "Build Params : --file ${buildDockerFile} ${appWorkingDir} "

            appCompileAndPacking = docker.build("${dockerRepo}/${dockerImageName}-build:${env.BUILD_NUMBER}","--file ${buildDockerFile} ${appWorkingDir}")
            def dockerCMD = readFile buildDockerLocation
            echo dockerCMD.substring(dockerCMD.indexof('CMD')+3,dockerCMD.length())
            appCompileAndPacking.inside {
            sh echo dockerCMD.substring(dockerCMD.indexof('CMD')+3,dockerCMD.length())
            } 
        }
}else {
        echo 'Compile and Package are not seperate steps , it is inferred that the package is not compiler dependant'
}
//____________________________________________________________________________________________________________________________________________________________________

    if (${stage}.toUpperCase() == 'BUILD'){
        echo 'It is inferred that the package is a Build only application , hence it is moved to a temporary repository'
        docker.WithRegistry("http://${DockerRegistryTemp}/",'docker-registry-login') {
                def buildImg
            stage ('Dockerization and Stage') {
            buildImg = docker.build("buildTagName","buildParams")
            buildImg.push();
            }
        }
    }else if ($(stage).toUpperCase() == 'DEPLOY'){
    echo 'It is inferred that the package is a deploy only application , hence it has to be moved to a permanent repository'
        docker.WithRegistry("http://${DockerRegistryPerm}/",'docker-registry-login'){
            def deployImg
            stage('Dockerization and Publish') {
            deployImg = docker.build("buildName","buildParams")
            deployImg.push('latest');
            }
        }
    }else if ($(stage).toUpperCase() == 'CERTIFY'){
    echo 'It is inferred that the package is a certify only application , hence it has to be moved to a provisioned with a runtime sandbox environment and push it to temporary repository'
        docker.WithRegistry("http://${DockerRegistryPerm}/","docker-registry-login"){
            def certifyImg
            stage ('Dockerization and Certify'){
            certifyImg = docker.build("buildName","buildParams")
            certifyImg.push('SNAPSHOT');
            }
        }        
    }

    stage('Publish the Jenkins Output to Nexus'){
        echo 'Publishing the Artifacts...'
        sh 'rm nexus.txt' 
    }
}
