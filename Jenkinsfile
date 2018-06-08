#!groovy

pipeline{
    agent any
    
    stages {
        stage('Build'){
            steps{
                bat 'xcopy "c:\test.txt" "c:\parent\subfolder1\" /z /i'
                bat 'xcopy "c:\test.txt" "c:\parent\subfolder2\" /z /i'
                bat 'xcopy "c:\test.txt" "c:\parent\subfolder3\" /z /i'
                bat 'xcopy "c:\test.txt" "c:\parent\subfolder4\" /z /i'
                bat 'xcopy "c:\test.txt" "c:\parent\subfolder5\" /z /i'
                
                echo 'Building Complete'
            }
        }
        stage('Test'){
            steps{
                echo 'Testing'
                echo 'Testing Complete'
            }
        }
        stage('Deploy'){
            steps{
                echo 'Deploying'
                echo 'Deploying Complete'
            }
        }
    }
}
