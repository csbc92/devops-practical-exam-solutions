# Pipeline

This assignment is more or less the result of [the ca-project](https://github.com/praqma-training/ca-project)

You are welcome to use your own project and set it up with the process described in the project.

## Requirements

* Pipeline should be _as code_ alongside the code
* The pipeline should have stages on at least two different nodes.
* It should be possible to download the newest artifact from Jenkins webpage
* All builds must be run in a docker container
* If at all possible, you should produce both the binaries, as well as a docker image on docker hub.
* You should use a pretested integration workflow (pull-request with build qualification, or the Pretested integration flow).

## Hand in

* Link to the running Jenkins server
* Link to the Git repository where the code is
* A description of your pipeline, and any problems that you run into during development.






# Solution

### github Repository
https://github.com/Vedsted/ca-project

### Jenkins build server
IP: 52.59.220.156:8080

user: admin

password: admin

## Problems
* Used port 80 by accident where Jenkins occupied the port..
* python interpreter version in the docker image, made the app.py unable to execute
* We have discussed whether or not to have the whole python application to be **a part of** the docker image, or if we should **volume in** the application (REF-1). The difference is, respectively, if the applications-files should be available at **build-time** or at **run-time**. For the unit-tests it was decided to volume into a base-image with python requirements, because this eliminates the need to build a new docker-image each time a developer commits a change. This allows a faster feedback loop. When the application needs to be testet at staging/production environment, it is desireble to actually make the application-files a part of the image, because this is the image that should be shipable upon release.
* We had a small pipeline up and running relatively easy. The hard part was to get the whole pipeline up and running from commit -> deploy into production. There was minimal changes all the time to the Jenkinsfile.

### Refs:

(REF-1) https://stackoverflow.com/questions/24958140/what-is-the-difference-between-the-copy-and-add-commands-in-a-dockerfile

### Pipeline description
We have built a `Jenkinsfile` with a groovy-script that carries out all the stages made. The stages are described roughly below:

* Preparation - Looks for branches matching the pattern \*\*/ready/\*\* and if so, the build server pulls the branch into the working direction.
* Unit-Tests - Pulls the latest base docker image with the required environment in order to run the tests. Then it starts a docker container and runs the tests.py script. 
* Push-VC - Utilizes the PretestedIntegrationPlugin from Praqma. The plugin merges the ready-branch into master if there is no merge-conflicts, using fast-forward.
* Deploy-test - Pulls the latest base docker image and then volumes the whole application into the container in order to start the web-server. We make a smoke-test by running curl on the URL where the web-server is hosted.
* Publish image - This is running a custom made deployment-script that builds the whole application into a new image so it is available at runtime. Then the script pushes the image to be available on https://hub.docker.com/r/vedsted/codechan/ .

Some of the stages are run of different nodes (for educational purposes), where publish-image stage has elevated priveleges in order to be able to push the image to docker hub.

The complete `Jenkinsfile` is shown below:

```groovy
node('non_privileged') {

    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        checkout([$class: 'GitSCM', branches: [[name: '**/ready/**']], 
        doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanBeforeCheckout'], 
        pretestedIntegration(gitIntegrationStrategy: accumulated(), integrationBranch: 'master', 
        repoName: 'origin')], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'thevinge', 
        url: 'git@github.com:Vedsted/ca-project.git']]])
    
    }
    
    stage('Unit-Tests') {
        // Run the tests
        if (isUnix()) {
         sh 'docker pull vedsted/codechan:latest-base'
         sh 'docker run -i --rm --name codechan_Test_Script -v $PWD:/usr/src/codechan -w /usr/src/codechan vedsted/codechan:latest-base python tests.py'

        }
    }
    stage('Push-VC'){
        pretestedIntegrationPublisher()
        stash name: "repo", includes: "**", useDefaultExcludes: false
        deleteDir()
    }

}

node('deployment_test'){
    stage('Deploy -test'){
        unstash 'repo'
        sh 'docker pull vedsted/codechan:latest-base'
        sh 'docker run -d --rm --name codechan_Test_Script -v $PWD:/usr/src/codechan -w /usr/src/codechan -p 5000:5000 vedsted/codechan:latest-base python run.py'
        sh 'sleep 8s'
        sh 'curl 127.0.0.1:5000'
        sh 'docker stop codechan_Test_Script'
        stash name: "repo_2", includes: "**", useDefaultExcludes: false
        deleteDir()
    }
}

node('privileged'){
	stage('Publish image'){
		unstash 'repo_2'
        sh 'cd deployment && ./deploy_image.sh'
        deleteDir()
    } 
}
```