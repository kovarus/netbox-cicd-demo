podTemplate(
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
        stage('Get Netbox Development Branch') {
            git branch: 'development', credentialsId: 'kovarus-github', url: 'https://github.com/kovarus/netbox-cicd-demo'
            container('python') {
                stage('Install Dependencies') {
                    sh 'pip install -r requirements.txt'
                    sh 'pip install pycodestyle'
                    sh 'apt-get update; apt-get install -y postgresql-client-9.6 redis-tools'
                }
                stage('Test Environment') {
                    sh 'psql --version'
                    sh 'psql -h localhost -U postgres -c \'SELECT version();\''
                    sh 'redis-cli ping'
                }
                stage('Run Tests on Development') {
                    sh './scripts/cibuild.sh'
                }
            }
        }
    }
}
