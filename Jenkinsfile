node('build-slave') {
    try {
        String ANSI_GREEN = "\u001B[32m"
        String ANSI_NORMAL = "\u001B[0m"
        String ANSI_BOLD = "\u001B[1m"
        String ANSI_RED = "\u001B[31m"
        String ANSI_YELLOW = "\u001B[33m"

         ansiColor('xterm') {
            stage('Checkout') {
                cleanWs()
                if(params.github_release_tag == ""){
                    checkout scm
                    commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    branch_name = sh(script: 'git name-rev --name-only HEAD | rev | cut -d "/" -f1| rev', returnStdout: true).trim()
                    build_tag = branch_name + "_" + commit_hash + "_" + env.BUILD_NUMBER
                    println(ANSI_BOLD + ANSI_YELLOW + "Tag not specified, using the latest commit hash: " + commit_hash + ANSI_NORMAL)
                }
                else {
                    def scmVars = checkout scm
                    checkout scm: [$class: 'GitSCM', branches: [[name: "refs/tags/$params.github_release_tag"]],  userRemoteConfigs: [[url: scmVars.GIT_URL]]]
                    build_tag = params.github_release_tag + "_" + env.BUILD_NUMBER
                    println(ANSI_BOLD + ANSI_YELLOW + "Tag specified, building from tag: " + params.github_release_tag + ANSI_NORMAL)
                }
                echo "build_tag: "+ build_tag
            }
        }

            stage('Build') {
                sh 'mvn clean install'
            }

            stage('Package') {
                sh "/opt/apache-maven-3.6.3/bin/mvn3.6 package -Pbuild-docker-image -Drelease-version=${build_tag}"
            }

            stage('Retagging'){
                sh """
                    docker tag secor:${build_tag} ${hub_org}/secor:${build_tag}
                    echo {\\"image_name\\" : \\"secor\\", \\"image_tag\\" : \\"${build_tag}\\", \\"node_name\\" : \\"${env.NODE_NAME}\\"} > metadata.json
                """
            }

            stage('Archive artifacts'){
                archiveArtifacts "metadata.json"
                currentBuild.description = "${build_tag}"
            }
        }

    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
