pipeline {
    agent any

    stages {
        stage('Build') {
            stages {
                stage('Compile') {
                    steps {
                        echo 'Compiling...'
                        sleep 10
                    }
                }
                stage('Package') {
                    steps {
                        echo 'Packaging...'
                        sleep 5
                    }
                }
            }
        }

        stage('Registering build artifact') {
            steps {
                echo 'Registering the metadata'
                echo 'Another echo to make the pipeline a bit more complex'
                registerBuildArtifactMetadata(
                    name: "artifacts-ninja-QA-comp-testing-000007",
                    version: "1.0.07",
                    type: "docker",
                    url: "http://localhost:0007",
                    digest: "6f637064707039346163663237383938",
                    label: "qa-ninja"
                )
            }
        }

        stage('Test') {
            steps {
                echo 'Running Unit Tests...'
                sleep 10
                echo 'Running Integration Tests...'
                sleep 5
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
                sleep 5
            }
        }
    }
}
