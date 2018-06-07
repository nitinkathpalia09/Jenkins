#!groovy

def versionPrefix = '1.0'
def versionNumber = "${versionPrefix}.${BUILD_NUMBER}.0"
def buildNumber = env.BUILD_NUMBER
def Version = null

//Build properties
properties([
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '15'))
])

try
{
    
   timestamps
    {
        node('redhat')
        {
            stage('Set Version')
            {
                Version = VersionNumber versionNumberString: "$versionNumber"
               
                echo(Version)
                //The display name needs to always be the version
                currentBuild.displayName = Version
                                
            }

            stage ('SCM Pull')
            {
                checkout scm
            }

            stage ('Compile')
            {
                sh 'scl enable devtoolset-7 "npm install"'
                sh 'npm install'
                sh 'grunt cibuild'
  
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
    def recipients = 'DataHub-DEP@thomsonreuters.com '
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"
    def details = """<p>${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""
  

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
