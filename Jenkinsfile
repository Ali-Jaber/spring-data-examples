pipeline {
    agent none

    triggers {
        pollSCM 'H/10 * * * *'
        upstream(upstreamProjects: "spring-data-cassandra/master,spring-data-elasticsearch/master,spring-data-gemfire/master,spring-data-j2dbc/master,spring-data-geode/master," +
                "spring-data-jpa/master,spring-data-ldap/master,spring-data-mongodb/master,spring-data-neo4j/master,spring-data-r2dbc/master,spring-data-redis/master,spring-data-rest/master," +
                "spring-data-solr/master",
                threshold: hudson.model.Result.SUCCESS)
    }

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage("Test") {
            parallel {
                stage("test: baseline") {
                    agent {
                        docker {
                            image 'adoptopenjdk/openjdk8:latest'
                            args '-v $HOME/.m2:/root/.m2'
                        }
                    }
                    steps {
                        sh "./mvnw clean dependency:list test -U -Dsort -Dmaven.test.redirectTestOutputToFile=true -B"
                    }
                }
            }
        }
    }

    post {
        changed {
            script {
                slackSend(
                        color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger',
                        channel: '#spring-data-dev',
                        message: "${currentBuild.fullDisplayName} - `${currentBuild.currentResult}`\n${env.BUILD_URL}")
                emailext(
                        subject: "[${currentBuild.fullDisplayName}] ${currentBuild.currentResult}",
                        mimeType: 'text/html',
                        recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                        body: "<a href=\"${env.BUILD_URL}\">${currentBuild.fullDisplayName} is reported as ${currentBuild.currentResult}</a>")
            }
        }
    }
}
