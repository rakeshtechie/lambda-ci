def bucket = 'deployment-packages-lambda-test'
def functionName = 'Fibonacci'
def region = 'ap-south-1'

node('slaves'){
    stage('Checkout'){
        checkout scm
    }

    stage('build npm'){
        sh 'npm install'
    }

    stage('Build'){
        sh "zip ${commitID()}.zip package.json node_modules index.js"
    }

    stage('Push'){
        sh "aws s3 cp ${commitID()}.zip s3://${bucket}"
    }

    stage('Deploy'){
        sh "aws lambda update-function-code --function-name ${functionName} \
                --s3-bucket ${bucket} \
                --s3-key ${commitID()}.zip \
                --region ${region}"
    }

    if (env.BRANCH_NAME == 'master') {
        stage('Publish') {
            def lambdaVersion = sh(
                script: "aws lambda publish-version --function-name ${functionName} --region ${region} | jq -r '.Version'",
                returnStdout: true
            )
            sh "aws lambda update-alias --function-name ${functionName} --name production --region ${region} --function-version ${lambdaVersion}"
        }
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}
