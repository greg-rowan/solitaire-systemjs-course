stage 'CI'
node {

    checkout scm

    //git branch: 'jenkins2-course', 
    //    url: 'https://github.com/g0t4/solitaire-systemjs-course'

    sh 'npm install phantomjs -g'
    sh 'npm install karma-phantomjs-launcher -g'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    sh 'npm run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
          
    //archiveArtifacts 'app/**'      
          
}


node('linux') {
    sh 'ls'
    sh 'rm -rf *'
    unstash 'everything'
    sh 'ls'
}

//parallel integration testing
//stage 'Browser Testing'
//parallel chrome: {
//    runTests("Chrome")
//}, firefox: {
//    runTests("Firefox")
//}
//, safari: {
//    runTests("Safari")
//}

def runTests(browser) {
    node {
        sh 'rm -rf *'

        unstash 'everything'

        // on windows use: bat "npm run test-single-run -- --browsers ${browser}"
        sh "npm run test-single-run -- --browsers ${browser}"

        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
    }
}
 
node {
    notify("Deploy to staging?")
}

// input step will pause pipeline, it should NOT be inside node block!
input 'Deploy to staging?'

stage name: 'Deploy', concurrency: 1
node{
    notify("Solitaire Deployed!")
}


def notify(status){
    emailext (
      to: "greg.rowan@macys.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}

