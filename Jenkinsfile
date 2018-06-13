!#groovy 
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                bat '''xcopy "c:\\test.txt" "c:\\parent\\subfolder1\\" /z /i
xcopy "c:\\test.txt" "c:\\parent\\subfolder2\\" /z /i
xcopy "c:\\test.txt" "c:\\parent\\subfolder3\\" /z /i
xcopy "c:\\test.txt" "c:\\parent\\subfolder4\\" /z /i
xcopy "c:\\test.txt" "c:\\parent\\subfolder5\\" /z /i'''
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
