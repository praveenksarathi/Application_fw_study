// This is a sample file for Multi-Platform integration Pipeline.
    def DockerRegsitryTemp = ''
    def DockerRegistryPerm = ''

node {
        stage 'Code Pickup From Repository' {
        echo 'The Picked Up Code Type is : ${CodeLocType}'
        echo 'The Picked Up Code Path is : ${CodeLocPath}'
        
        if ("${CodeLocType}").toUpperCase() == 'GIT' {
        } else if ("${CodeLocType}").toUpperCase() == 'SVN' {
        }else
        {
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
}else {
        echo 'No Test Modules found , hence it is assumed that no test environment and / or test cases to be performed'
        testPath = ''
}

// Build And Packaging

def buildAndPackagingReq = True
def buildDockerLocation = appPath + 'Dockerfile.build'
def distDockerLocation = appPath + 'Dockerfile.dist'
def singleDockerFile = appPath + 'Dockerfile'

if (fileExists(buildDockerLocation) && fileExists(distDockerLocation)) {
        echo 'Looks like this application contains seperate Build and Distribution Docker Files , its assumed to be developed in compiler dependant programming language.'
        buildAndPackagingReq = True
} else if (fileExists(singleDockerFile)) {
        echo 'This application contains a single docker file , it is assumed to be developed in compiler in-dependant / interpreter based programming language.'
        buildAndPackagingReq = False
        distDockerFile = singleDockerFile
} else {
        error 'No Docker File found in the specified path :' + appPath
        buildAndPackagingReq = False
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
                def buildTagName = '${dockerRepo}/${dockerImageName}-build:${env.BUILD_NUMBER}'
                def buildParams =  '--file ${buildDockerFile} ${appWorkingDir}'
                echo "Build Tag Name :" +buildTagName
                echo "Build Params : " +buildParams

                appCompileAndPacking = docker.build("buildTagName","buildParams")
        }
}else {
        echo 'Compile and Package are not seperate steps , it is inferred that the package is not compiler dependant'
}
// *** LEFT EDITING HERE **//




}
