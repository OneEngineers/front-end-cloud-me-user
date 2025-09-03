pipeline {
    agent any

    environment {
        REGISTRY        = "reaksa7236/lms-report"
        DOCKER_REGISTRY = "docker.io"
        DOCKER_USER     = credentials('docker-username')   // Jenkins credential ID
        DOCKER_PASS     = credentials('docker-password')   // Jenkins credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Detect Branch/Tag') {
            steps {
                script {
                    // Detect if running on tag or branch
                    if (env.GIT_BRANCH ==~ /.*tags\/.*/) {
                        TAG = env.GIT_BRANCH.replaceAll('refs/tags/', '')
                        IS_TAG = true
                    } else {
                        TAG = env.BRANCH_NAME
                        IS_TAG = false
                    }
                    echo "Running build for: ${TAG} (isTag=${IS_TAG})"

                    // Determine ENV and VERSION
                    if (IS_TAG) {
                        if (TAG.startsWith("dev-v")) {
                            VERSION = TAG.replace("dev-v", "")
                            ENV = "dev"
                        } else if (TAG.startsWith("stg-v")) {
                            VERSION = TAG.replace("stg-v", "")
                            ENV = "stg"
                        } else if (TAG.startsWith("prod-v")) {
                            VERSION = TAG.replace("prod-v", "")
                            ENV = "prod"
                        } else {
                            error("Invalid tag format: ${TAG}")
                        }
                    } else {
                        // Handle branches
                        if (TAG == "develop") {
                            VERSION = "latest"
                            ENV = "dev"
                        } else if (TAG == "staging") {
                            VERSION = "latest"
                            ENV = "stg"
                        } else if (TAG == "main" || TAG == "master") {
                            VERSION = "latest"
                            ENV = "prod"
                        } else {
                            error("Branch ${TAG} not configured for builds")
                        }
                    }

                    echo "Extracted VERSION=${VERSION}, ENV=${ENV}"
                }
            }
        }

        stage('Docker Login') {
            steps {
                sh """
                  echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin ${DOCKER_REGISTRY}
                """
            }
        }

        stage('Build & Push Docker') {
            steps {
                script {
                    if (ENV == "dev") {
                        sh """
                          docker build -t ${REGISTRY}:dev-${VERSION} .
                          docker push ${REGISTRY}:dev-${VERSION}
                          docker build -t ${REGISTRY}:master .
                          docker push ${REGISTRY}:master
                        """
                    } else if (ENV == "stg") {
                        sh """
                          docker build -t ${REGISTRY}:stg-${VERSION} .
                          docker push ${REGISTRY}:stg-${VERSION}
                          docker build -t ${REGISTRY}:testing .
                          docker push ${REGISTRY}:testing
                        """
                    } else if (ENV == "prod") {
                        sh """
                          docker build -t ${REGISTRY}:prod-${VERSION} .
                          docker push ${REGISTRY}:prod-${VERSION}
                          docker build -t ${REGISTRY}:production .
                          docker push ${REGISTRY}:production
                        """
                    }
                }
            }
        }
    }
}
