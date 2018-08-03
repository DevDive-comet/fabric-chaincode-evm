// Copyright IBM Corp All Rights Reserved
//
// SPDX-License-Identifier: Apache-2.0
//
node ('hyp-x') { // trigger build on x86_64 node
     def ROOTDIR = pwd() // workspace dir (/w/workspace/<job_name>
     env.PROJECT_DIR = "gopath/src/github.com/hyperledger"
     env.GOPATH = "$WORKSPACE/gopath"
     env.JAVA_HOME = "/usr/lib/jvm/java-1.8.0-openjdk-amd64"
     env.PATH = "$GOPATH/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:~/npm/bin:/home/jenkins/.nvm/versions/node/v6.9.5/bin:/home/jenkins/.nvm/versions/node/v8.9.4/bin:$PATH"
     env.GOROOT = "/opt/go/go1.10.linux.amd64"
     env.PATH = "$GOROOT/bin:$PATH"

     def failure_stage = "none"
// delete working directory
     deleteDir()
      stage("Fetch Patchset") { // fetch gerrit refspec on latest commit
          try {
              dir("${ROOTDIR}"){
              sh '''
                 [ -e gopath/src/github.com/hyperledger/fabric-chaincode-evm ] || mkdir -p $PROJECT_DIR
                 cd $PROJECT_DIR
                 git clone git://cloud.hyperledger.org/mirror/fabric-chaincode-evm && cd fabric-chaincode-evm
                 git checkout "$GERRIT_BRANCH" && git fetch origin "$GERRIT_REFSPEC" && git checkout FETCH_HEAD
              '''
              }
          }
          catch (err) {
                 failure_stage = "Fetch patchset"
                 throw err
          }
      }
// clean environment, get env data
      stage("CleanEnv - GetEnv") {
          try {
                 dir("${ROOTDIR}/$PROJECT_DIR/fabric-chaincode-evm/scripts/jenkins_scripts") {
                 sh './CI_Script.sh --clean_Environment --env_Info'
                 }
          }
          catch (err) {
                 failure_stage = "Clean Environment - Get Env Info"
                 throw err
          }
      }

// Build Build Images
      stage("Build-Images") {
          try {
                 dir("${ROOTDIR}") {
                 sh '''
                    [ -e gopath/src/github.com/hyperledger/fabric ] || mkdir -p $PROJECT_DIR
                    cd $PROJECT_DIR
                    git clone git://cloud.hyperledger.org/mirror/fabric && cd fabric
                    git checkout "$GERRIT_BRANCH"
                    make buildenv
                    cd ../fabric-chaincode-evm && make docker
                 '''
                 }
          }
          catch (err) {
                 failure_stage = "build images"
                 throw err
          }
      }

// Run license-checks
      stage("Checks") {
          try {
                 dir("${ROOTDIR}/$PROJECT_DIR/fabric-chaincode-evm") {
                 sh '''
                    echo "------> Run license checks"
                    make license
                 '''
                 }
          }
          catch (err) {
                 failure_stage = "license"
                 throw err
          }
      }

// Run unit-tests (unit-tests)
      stage("Unit-Tests") {
          try {
                 dir("${ROOTDIR}/$PROJECT_DIR/fabric-chaincode-evm") {
                 sh '''
                    echo "------> Run unit-tests"
                    make unit-tests
                 '''
                 }
          }
          catch (err) {
                 failure_stage = "unit-tests"
                 throw err
          }
      }
// Run integration tests (e2e tests)
      stage("Integration-Tests") {
          try {
                 dir("${ROOTDIR}/$PROJECT_DIR/fabric-chaincode-evm") {
                 sh '''
                    echo "-------> Run integration-tests"
                    make integration-test
                 '''
                 }
          }
          catch (err) {
                 failure_stage = "integration-test"
                 throw err
          }
      }
} // node block end here
