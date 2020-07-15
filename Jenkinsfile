properties([
    parameters([
        string(defaultValue: '', description: '', name: 'env_identifier', trim: false)
    ])
])
podTemplate(
    annotations: [podAnnotation(key: 'iam.amazonaws.com/role', value: "${env.IAM_ARN_GET_TAGS}")],
    containers: [
        containerTemplate(
            name: 'postgres',
            image: 'postgres:9.6',
            ttyEnabled: true,
            command: '/usr/local/bin/docker-entrypoint.sh',
            args: 'postgres',
            workingDir: '/home/jenkins/agent/netbox',
            envVars: [
              envVar(key: 'POSTGRES_PASSWORD', value: 'test')
            ]
        ),
        containerTemplate(
            name: 'redis',
            image: 'redis:6',
            ttyEnabled: true,
            command: '/usr/local/bin/docker-entrypoint.sh',
            args: 'redis-server',
            workingDir: '/home/jenkins/agent/netbox'
        ),
        containerTemplate(
            name: 'python',
            image: 'python:3.5-stretch',
            command: 'cat',
            ttyEnabled: true,
            workingDir: '/home/jenkins/agent/netbox',
        )
    ]
){
    node(POD_LABEL) {
        ws('development') {
            stage('Get Netbox Development Branch') {
                checkout([
                    $class: 'GitSCM',
                    branches: [
                        [name: '*/development']
                    ],
                    doGenerateSubmoduleConfigurations: false,
                    userRemoteConfigs: [
                        [credentialsId: 'kovarus-github', url: 'https://github.com/kovarus/netbox-cicd-demo']
                    ]
                ])
                container('python') {
                    stage('Install Dependencies') {
                        // sh 'pip install --no-cache-dir -r requirements.txt'
                        // sh 'pip install --no-cache-dir pycodestyle'
                        sh 'apt-get update; apt-get install -y postgresql-client-9.6 redis-tools awscli'
                    }
                    stage('Test Environment') {
                        // sh 'psql --version'
                        // sh 'psql -h localhost -U postgres -c \'SELECT version();\''
                        // sh 'redis-cli ping'
                    }
                    stage('Run Tests on Development') {
                        // sh './scripts/cibuild.sh'
                    }
                    stage('Clean Up') {
                        // sh 'pip freeze | xargs pip uninstall -y'
                    }
                }
            }
        }
        stage('Merge into Master') {
            ws('master') {
                checkout([
                    $class: 'GitSCM',
                    branches: [
                        [name: '*/development']
                    ],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [
                        [$class: 'PreBuildMerge', options: [mergeRemote: 'origin', mergeTarget: 'master']],
                        [$class: 'UserIdentity', email: 'kpsc-automation@users.noreply.github.com', name: 'kpsc-automation']
                    ],
                    userRemoteConfigs: [
                        [credentialsId: 'kovarus-github', url: 'https://github.com/kovarus/netbox-cicd-demo']
                    ]
                ])
                container('python') {
                    stage('Install Dependencies') {
                        // sh 'pip install --no-cache-dir -r requirements.txt'
                        // sh 'pip install --no-cache-dir pycodestyle'
                    }
                    stage('Run Tests on Master') {
                        // sh './scripts/cibuild.sh'
                    }
                    stage('Tag and Push to Master') {
                        GIT_TAG = sh script: 'git describe --tags | awk -F\'[.]\' \'{print $1"."$2"."$3+1}\'', returnStdout: true
                        // sh "git tag ${GIT_TAG}"
                        // withCredentials([usernamePassword(credentialsId: 'kovarus-github', passwordVariable: 'GITHUB_PASSWORD', usernameVariable: 'GITHUB_USERNAME')]) {
                        //     sh "git push --tags https://${GITHUB_USERNAME}:${GITHUB_PASSWORD}@github.com/kovarus/netbox-cicd-demo HEAD:master"
                        // }
                    }
                    stage('Get Versions and Active Side') {
                        ACTIVE_SIDE = sh script: "aws ec2 describe-instances --region us-west-2 --filters \"Name=tag:Name,Values=nb-rc*${params.env_identifier}*\" --query \"Reservations[*].Instances[*].Tags[?Key=='Active'].Value[]\" --output text", returnStdout: true
                        if(ACTIVE_SIDE == 'Green') {
                            DEPLOY_TO = 'Blue'
                        } else if(ACTIVE_SIDE == 'Blue') {
                            DEPLOY_TO = 'Green'
                        } else {
                            DEPLOY_TO = 'Unknown'
                        }
                        writeFile file: 'deploy_to.out', text: "${DEPLOY_TO}".trim()
                        archiveArtifacts artifacts: 'deploy_to.out', followSymlinks: false

                        BLUE_VER = sh script: "aws ec2 describe-instances --region us-west-2 --filters \"Name=tag:Name,Values=nb-rc*${params.env_identifier}*\" --query \"Reservations[*].Instances[*].Tags[?Key=='BlueVersion'].Value[]\" --output text", returnStdout: true
                        writeFile file: 'blue_version.out', text: "${BLUE_VER}".trim()
                        archiveArtifacts artifacts: 'blue_version.out', followSymlinks: false

                        GREEN_VER = sh script: "aws ec2 describe-instances --region us-west-2 --filters \"Name=tag:Name,Values=nb-rc*${params.env_identifier}*\" --query \"Reservations[*].Instances[*].Tags[?Key=='GreenVersion'].Value[]\" --output text", returnStdout: true
                        writeFile file: 'green_version.out', text: "${GREEN_VER}".trim()
                        archiveArtifacts artifacts: 'green_version.out', followSymlinks: false
                    }
                }
            }
        }
    }
}