pipeline {
    agent any

    environment {
        CODEQL_URL = "https://github.com/github/codeql-action/releases/latest/download/codeql-bundle-linux64.tar.gz"
        CODEQL_DIR = "${env.WORKSPACE}/codeql"
        SOURCE_DIR = "${env.WORKSPACE}/test-go-project"
        DB_NAME = "my-app-db-1"
        SARIF_OUTPUT = "result1.sarif"
        GO_VERSION = "1.21.5"
        GO_URL_X86 = "https://go.dev/dl/go1.21.5.linux-amd64.tar.gz"
        GO_URL_ARM = "https://go.dev/dl/go1.21.5.linux-arm64.tar.gz"  // ARM64 URL
        GO_DIR = "${env.WORKSPACE}/go"
        GOROOT = "${env.WORKSPACE}/go"
        GOPATH = "${env.WORKSPACE}/go-packages"
        PATH = "${env.WORKSPACE}/go/bin:${env.PATH}"
    }

    stages {
        stage('Install Go') {
            steps {
                echo "⬇️ Installing Go..."
                script {
                    def tempDir = sh(script: 'mktemp -d', returnStdout: true).trim()
                    def arch = sh(script: 'uname -m', returnStdout: true).trim()
                    
                    def goUrl
                    if (arch == 'aarch64') {
                        goUrl = GO_URL_ARM
                        echo "Installing Go for ARM architecture (aarch64)..."
                    } else if (arch == 'x86_64') {
                        goUrl = GO_URL_X86
                        echo "Installing Go for x86_64 architecture..."
                    } else {
                        error "Unsupported architecture: ${arch}. This pipeline supports x86_64 and aarch64 only."
                    }

                    // Download and extract the appropriate Go binary
                    echo "⬇️ Downloading Go from ${goUrl}..."
                    sh """
                        curl -LO ${goUrl}
                        if [[ ! -f go1.21.5.linux-${arch}.tar.gz ]]; then
                            echo "Error: Go tarball not found."
                            exit 1
                        fi
                        tar -xzf go1.21.5.linux-${arch}.tar.gz -C ${tempDir}
                        rm -rf ${GO_DIR}
                        mv ${tempDir}/go ${GO_DIR}
                        echo "✅ Go installed at ${GO_DIR}"
                    """
                }
            }
        }

        stage('Check System Architecture') {
            steps {
                script {
                    def arch = sh(script: 'uname -m', returnStdout: true).trim()
                    if (arch != 'x86_64' && arch != 'aarch64') {
                        error "⚠️ Unsupported architecture: ${arch}. This pipeline requires x86_64 or aarch64 architecture."
                    }
                    echo "✅ Architecture is supported: ${arch}"
                }
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

        stage('Install Dependencies') {
            steps {
                echo "⬇️ Installing necessary dependencies..."
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y libc6 lib32gcc1
                    echo "✅ Dependencies installed"
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
