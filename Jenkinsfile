node {
    try{
        
        //bitbucketStatusNotify(buildState: 'INPROGRESS')
        
        ws("${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}/") {
            withEnv(["GOPATH=${JENKINS_HOME}/jobs/${JOB_NAME}/builds/${BUILD_ID}"]) {
                env.PATH="${GOPATH}/bin:$PATH"
                
                stage('Checkout'){
                    echo 'Checking out SCM'
                    checkout scm
                }
                
                stage('Pre Test'){
                    echo 'Pulling Dependencies'
            
                    sh 'go version'
                    /*sh 'go get -u github.com/golang/dep/cmd/dep'
                    sh 'go get -u github.com/golang/lint/golint'
                    sh 'go get github.com/tebeka/go2xunit'*/
                    
                    //or -update
                    sh 'cd ${GOPATH}/src/cmd/project/ && dep ensure' 
                }
        
                stage('Test'){
                    
                    //List all our project files with 'go list ./... | grep -v /vendor/ | grep -v github.com | grep -v golang.org'
                    //Push our project files relative to ./src
                    sh 'cd $GOPATH && go list ./... | grep -v /vendor/ | grep -v github.com | grep -v golang.org > projectPaths'
                    
                    //Print them with 'awk '$0="./src/"$0' projectPaths' in order to get full relative path to $GOPATH
                    def paths = sh returnStdout: true, script: """awk '\$0="./src/"\$0' projectPaths"""
                    
                    echo 'Vetting'

                    sh """cd $GOPATH && go tool vet ${paths}"""

                    echo 'Linting'
                    sh """cd $GOPATH && golint ${paths}"""
                    
                    echo 'Testing'
                    sh """cd $GOPATH && go test -race -cover ${paths}"""
                }
            
                stage('Build'){
                    echo 'Building Executable'
                
                    //Produced binary is $GOPATH/src/cmd/project/project
                    sh """cd $GOPATH/src/cmd/project/ && go build -ldflags '-s'"""
                }
                
                /*stage('BitBucket Publish'){
                
                    //Find out commit hash
                    sh 'git rev-parse HEAD > commit'
                    def commit = readFile('commit').trim()
                
                    //Find out current branch
                    sh 'git name-rev --name-only HEAD > GIT_BRANCH'
                    def branch = readFile('GIT_BRANCH').trim()
                    
                    //strip off repo-name/origin/ (optional)
                    branch = branch.substring(branch.lastIndexOf('/') + 1)
                
                    def archive = "${GOPATH}/project-${branch}-${commit}.tar.gz"

                    echo "Building Archive ${archive}"
                    
                    sh """tar -cvzf ${archive} $GOPATH/src/cmd/project/project"""

                    echo "Uploading ${archive} to BitBucket Downloads"
                    withCredentials([string(credentialsId: 'bb-upload-key', variable: 'KEY')]) { 
                        sh """curl -s -u 'user:${KEY}' -X POST 'downloads-page-url' --form files=@'${archive}' --fail"""
                    }
                }*/
            }
        }
    }catch (e) {
        // If there was an exception thrown, the build failed
        currentBuild.result = "FAILED"
        
        //bitbucketStatusNotify(buildState: 'FAILED')
    } finally {
        // Success or failure, always send notifications
        //notifyBuild(currentBuild.result)
        
        def bs = currentBuild.result ?: 'SUCCESSFUL'
        /*if(bs == 'SUCCESSFUL'){
            bitbucketStatusNotify(buildState: 'SUCCESSFUL')
        }*/
    }
}