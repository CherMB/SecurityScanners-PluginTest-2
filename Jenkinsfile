pipeline {
    agent any

    environment {
        CODEQL_URL = "https://github.com/github/codeql-action/releases/latest/download/codeql-bundle-linux64.tar.gz"
        CODEQL_DIR = "${env.WORKSPACE}/codeql"
        SOURCE_DIR = "${env.WORKSPACE}/test-go-project"
        DB_NAME = "my-app-db-1"
        SARIF_OUTPUT = "result1.sarif"
        GO_VERSION = "1.21.5"
        GO_DIR = "${env.WORKSPACE}/go"
        GOROOT = "${env.WORKSPACE}/go"
        GOPATH = "${env.WORKSPACE}/go-packages"
        PATH = "${env.WORKSPACE}/go/bin:${env.PATH}"
    }

    stages {
        stage('Install Go') {
            steps {
                echo "⬇️ Installing Go..."
                sh '''
                    export GO_TEMP_DIR=$(mktemp -d)
                    curl -LO "https://go.dev/dl/go1.21.5.linux-amd64.tar.gz"
                    tar -xzf go1.21.5.linux-amd64.tar.gz -C "$GO_TEMP_DIR"
                    rm -rf "$GO_DIR"
                    mv "$GO_TEMP_DIR/go" "$GO_DIR"
                    echo "✅ Go installed at $GO_DIR"
                '''
            }
        }

        stage('Download and Extract CodeQL') {
            steps {
                echo "⬇️ Downloading CodeQL bundle..."
                sh '''
                    mkdir -p "$CODEQL_DIR"
                    curl -L "$CODEQL_URL" -o codeql-bundle.tar.gz
                    tar -xzf codeql-bundle.tar.gz -C "$CODEQL_DIR" --strip-components=1
                    echo "✅ CodeQL installed to $CODEQL_DIR"
                '''
            }
        }

        stage('Create CodeQL Database') {
            steps {
                echo "📦 Creating CodeQL database from source..."
                sh '''
                    rm -rf "$DB_NAME"
                    export PATH="$GO_DIR/bin:$PATH"
                    export GOROOT="$GO_DIR"
                    export GOPATH="$GOPATH"
                    "$CODEQL_DIR/codeql" database create "$DB_NAME" \
                      --language=go \
                      --source-root="$SOURCE_DIR"
                '''
            }
        }

        stage('Analyze Code with CodeQL') {
            steps {
                echo "🔍 Running CodeQL analysis..."
                sh '''
                    "$CODEQL_DIR/codeql" database analyze "$DB_NAME" \
                      codeql/go-queries \
                      --format=sarifv2.1.0 \
                      --output="$SARIF_OUTPUT"
                '''
            }
        }

        stage('Publish SARIF to Dashboard') {
            steps {
                echo "📄 Publishing SARIF Report..."
                archiveArtifacts artifacts: "$SARIF_OUTPUT", allowEmptyArchive: true, fingerprint: true
            }
        }
    }

    post {
        always {
            echo "✅ Build completed"
        }
    }
}
 