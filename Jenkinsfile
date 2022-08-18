import hudson.FilePath;
pipeline{
    /* agent { label 'master' } */
    agent { label 'windows_slave' }
    environment{
        dockerHub = 'https://harbor.dev.cloudsprint.io'
        imageRepo = 'harbor.dev.cloudsprint.io'
        imageName = 'fego-apps/nerddinner'
        imageRepoUser = 'fegouser'
        imageRepoPwd = 'enteryourpwd'
        acrName = 'cogacr'
        resourceGroup = 'aks-windows'
        clusterName = 'aks-windows'
        loginServer = "harbor.dev.cloudsprint.io"
        imageWithTag = "$imageRepo/$imageName:latest"

    }
    /* tools{
        maven 'MVN3'
    } */
    stages{

    stage('Build') {
        steps {
            bat 'echo'
            bat 'echo "Initial directory: "'
            bat 'dir'
            script {
                // Find all
                // Find all files with name "Dockerfile"
                def files = findFiles(glob: '**/Dockerfile')
                echo "Dockerfile files - $files"

                // Loop through all dockerfile paths and then build the dotnet project
                for (i = 0; i < files.size(); i++) {
                    //Build the project
                    echo "Building application ${i+1} at path -> ${files[i].path}"

                    // Get the directory of the dockerfile (removing the filename)

                    def dockerfilePathAsString = env.WORKSPACE + "\\" + "${files[i].path}"
                    echo "dockerfilePathAsString - ${dockerfilePathAsString}"

                    //Convert the file path in Windows format
                    def dockerfilePathAsStringInWindowsFormat = dockerfilePathAsString.replace("/", "\\")
                    echo "dockerfilePathAsStringInWindowsFormat - ${dockerfilePathAsStringInWindowsFormat}"

                    //Convert the string as a file object
                    def dockerfilePathAsFile = new File(dockerfilePathAsStringInWindowsFormat)
                    echo "dockerfilePathAsFile - ${dockerfilePathAsFile}"

                    // Get the filepath from the file object
                    def dockerfilepathnamewithoutparent = new FilePath(dockerfilePathAsFile)
                    echo "dockerfilepathnamewithoutparent - ${dockerfilepathnamewithoutparent}"

                    //Get the parent
                    def dockerfileDirectory = dockerfilepathnamewithoutparent.parent
                    echo "dockerfileDirectory - ${dockerfileDirectory}"

                    dir ("$dockerfileDirectory")
                    {
                        bat 'dir'
                        bat 'echo "...... Building the project ${dockerfileDirectory} ......"'
                        bat 'nuget restore ./../NerdApp.sln'
                        echo "Done restoring nuget packages for app ${i+1}."
                        bat 'msbuild ./../NerdApp.sln'
                        echo "Done building app ${i+1}."
                        bat 'echo'
                    }
                }
                echo 'Done building all apps.'
            }
        }
    }

        stage('deploy'){
            steps{
                withCredentials([azureServicePrincipal(credentialsId: 'aks-windows',
                    subscriptionIdVariable: 'SUBS_ID',
                    clientIdVariable: 'CLIENT_ID',
                    clientSecretVariable: 'CLIENT_SECRET',
                    tenantIdVariable: 'TENANT_ID')]) {
                        bat '''
                            az login --service-principal -u %CLIENT_ID% -p %CLIENT_SECRET% -t %TENANT_ID%
                            az aks get-credentials --name $clusterName --resource-group $resourceGroup

                            docker login -u $imageRepoUser -p $imageRepoPwd $imageRepo



                        '''
                        script{
                            def image = docker.build(imageWithTag,"./src")
                            image.push()
                            bat '''
                                kubectl create secret docker-registry regcred --docker-server=$imageRepo --docker-username=$imageRepoUser --docker-password=$imageRepoPwd                            
                                kubectl apply -f Nerd.yaml
                                az logout
                                docker logout $loginServer
                            '''
                        }
                    }
            }

        }
    }

}
