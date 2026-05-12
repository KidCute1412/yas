import groovy.transform.Field

@Field def SERVICE_PARAM_MAP = [
    cart            : 'TAG_CART',
    tax             : 'TAG_TAX',
    order           : 'TAG_ORDER',
    product         : 'TAG_PRODUCT',
    media           : 'TAG_MEDIA',
    rating          : 'TAG_RATING',
    customer        : 'TAG_CUSTOMER',
    location        : 'TAG_LOCATION',
    inventory       : 'TAG_INVENTORY',
    search          : 'TAG_SEARCH',
    payment         : 'TAG_PAYMENT',
    promotion       : 'TAG_PROMOTION',
    recommendation  : 'TAG_RECOMMENDATION',
    sampledata      : 'TAG_SAMPLEDATA',
    webhook         : 'TAG_WEBHOOK',
    'backoffice-bff': 'TAG_BACKOFFICE_BFF',
    'storefront-bff': 'TAG_STOREFRONT_BFF',
    'backoffice-ui' : 'TAG_BACKOFFICE_UI',
    'storefront-ui' : 'TAG_STOREFRONT_UI'
]

@Field def SERVICE_ALIASES = [
    'backoffice-ui' : ['backoffice-ui', 'backoffice'],
    'storefront-ui' : ['storefront-ui', 'storefront']
]

@Field def DOCKERFILE_PATTERNS = [
    '%s/Dockerfile',
    'services/%s/Dockerfile',
    'src/%s/Dockerfile',
    'apps/%s/Dockerfile',
    'applications/%s/Dockerfile'
]

pipeline {
    agent { label 'build-agent' }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        DOCKERHUB_NAMESPACE = 'thanhtienntt'
        DOCKERHUB_CREDS = 'dockerhub-creds'
        MAIN_BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Image Tag') {
            steps {
                script {
                    env.COMMIT_ID = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    if (env.BRANCH_NAME == env.MAIN_BRANCH) {
                        env.IMAGE_TAG = 'latest'
                    } else {
                        env.IMAGE_TAG = env.COMMIT_ID
                    }

                    echo "Branch name : ${env.BRANCH_NAME}"
                    echo "Commit id   : ${env.COMMIT_ID}"
                    echo "Image tag   : ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Java Services') {
            steps {
                sh '''
                    if [ -f "./mvnw" ]; then
                        chmod +x ./mvnw
                        ./mvnw clean package -DskipTests
                    else
                        mvn clean package -DskipTests
                    fi
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Build and Push All Services') {
            steps {
                script {
                    SERVICE_PARAM_MAP.each { serviceName, paramName ->
                        def candidateNames = SERVICE_ALIASES.containsKey(serviceName)
                            ? SERVICE_ALIASES[serviceName]
                            : [serviceName]

                        def dockerConfig = null

                        candidateNames.each { candidateName ->
                            DOCKERFILE_PATTERNS.each { pattern ->
                                if (dockerConfig == null) {
                                    def dockerfilePath = String.format(pattern, candidateName)

                                    if (fileExists(dockerfilePath)) {
                                        def contextPath = dockerfilePath.substring(0, dockerfilePath.lastIndexOf('/'))

                                        dockerConfig = [
                                            dockerfile: dockerfilePath,
                                            context   : contextPath
                                        ]
                                    }
                                }
                            }
                        }

                        if (dockerConfig == null) {
                            error 
                            """
                              Cannot find Dockerfile for service: ${serviceName}

                              Checked names:
                              ${candidateNames}

                              Checked patterns:
                              ${DOCKERFILE_PATTERNS}

                              Please update SERVICE_ALIASES or DOCKERFILE_PATTERNS in Jenkinsfile.
                            """
                        }

                        def imageName = "${DOCKERHUB_NAMESPACE}/yas-${serviceName}:${IMAGE_TAG}"

                        sh """
                            echo "=========================================="
                            echo "Building service : ${serviceName}"
                            echo "Image            : ${imageName}"
                            echo "Dockerfile       : ${dockerConfig.dockerfile}"
                            echo "Context          : ${dockerConfig.context}"
                            echo "=========================================="

                            docker build \
                              -f "${dockerConfig.dockerfile}" \
                              -t "${imageName}" \
                              "${dockerConfig.context}"

                            docker push "${imageName}"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }

        success {
            echo "All images were built and pushed successfully with tag: ${env.IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed. Check Dockerfile paths, Docker daemon, or Docker Hub credentials."
        }
    }
}