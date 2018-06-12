#!groovy

def versionPrefix = '1.0'
def versionNumber = "${versionPrefix}.${BUILD_NUMBER}.0"
def buildNumber = env.BUILD_NUMBER
def Version = null

//Build properties
properties([
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '30'))
])

try
{
    
   timestamps
    {
        node('Win_EikonTopNews')
        {
            env.JAVA_HOME="${tool 'jdk-8u65'}"
            env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"
            sh 'java -version'
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '<CREDENTIAL_ID>',
                    usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {

            println(env.USERNAME)
            
            stage('Set Version')
            {
                Version = VersionNumber versionNumberString: "$versionNumber"
               
                echo(Version)
                //The display name needs to always be the version
                currentBuild.displayName = Version
                                
            }

            stage ('SCM Pull')
            {
                 steps {
                git branch: '*/feature/cognizant/gil_ci',
                url: 'http://tfstta.int.thomson.com:8080/tfs/defaultcollection/OneSourceProvision/_git/OTP_GIL_UI'

                sh "ls -lat"
     
            }
            }
            stage ('Compile')
            {
               bat '''
                DEL "*.zip" 
                cd UI
                npm install --save-dev browser-sync-webpack-plugin
                cd UI
                npm run build:dev
                '''            
            }

            // stage('Package Artifacts')
            // {
            //     sh 'grunt postbuild'
            // }

            stage ('Upload to BAMS')
            {
                echo 'Pushing to BAMS'
                try
                {
                    dir('build')
                    {
                        def server = Artifactory.server('BAMS')
                        def uploadSpec = """{
                            "files": [
                            {
                            "pattern": "*.zip",
                            "target": "default.generic.local/trta-dep-datahub/datahub-service-masterdata/${BRANCH_NAME}/",                   
                            "recursive": "true",
                            "flat": "false",
                            "props": "Branch=${BRANCH_NAME};Version=$Version"                    
                            }
                            ]
                            }"""
    
    
                        def buildInfo = Artifactory.newBuildInfo()
                		server.upload spec: uploadSpec, buildInfo: buildInfo
                		server.publishBuildInfo buildInfo
    
                    }
                }catch(e)
                {
                    
                    if ((e.getMessage() == 'Rejected: org.jfrog.build.api.Artifact') || (e.getMessage().contains('remote file operation failed:')))
                    { 
                        echo 'Artifactory Plugin error, continue' 
                        currentBuild.result = 'SUCCESS'
                    }
                    else 
                    {
                        echo e.getMessage()
                        currentBuild.result = 'FAILURE'
                    }
                }
                    
                }
                    
            }
               
        } 

    }catch (e)
    {
        echo e.getMessage()
        currentBuild.result = 'FAILURE'

    } finally
    {
        notifyBuild(currentBuild.result)
    }
    
    def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus =  buildStatus ?: 'SUCCESS'

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def recipients = 'TF-TFPE-ReleaseMgmt@thomsonreuters.com,ProvisionGILWeb@thomsonreuters.com,Provision.DevOPS@thomsonreuters.com'
    def subject = "$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!"
    def summary = "${JELLY_SCRIPT, template="ooxp"}."
   
  

    // Override default values based on build status
    if (buildStatus == 'UNSTABLE') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

        // Send email
        emailext (
            subject: subject,
            body: details,
            to: recipients
            )
   
}
