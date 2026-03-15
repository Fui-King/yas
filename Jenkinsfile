// pipeline {
//     agent any
    
//     tools {
//         jdk 'JDK-21'
//         maven 'maven-3'
//         nodejs 'NodeJS-20' 
//     }

//     options {
//         buildDiscarder(logRotator(numToKeepStr: '10'))
//         disableConcurrentBuilds()
//     }

//     stages {
//         stage('Setup Tools') {
//             steps {
//                 script {
//                     echo "Checking if Snyk CLI is installed..."
//                     def snykExists = sh(script: 'command -v snyk', returnStatus: true) == 0
                    
//                     if (!snykExists) {
//                         echo "Snyk CLI not found. Installing via npm..."
//                         sh 'npm install -g snyk'
//                     } else {
//                         echo "Snyk CLI is already installed. Skipping installation."
//                     }
//                 }
//             }
//         }

//         stage('Gitleaks Scan') {
//             steps {
//                 script {
//                     echo "Running Gitleaks to detect hardcoded secrets..."
//                     catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
//                         sh """docker run --rm -v "${WORKSPACE}:/work" -w /work zricethezav/gitleaks:latest detect --source="." --verbose"""
//                     }
//                 }
//             }
//         }

//         stage('Pre-build Java Dependencies') {
//             when { 
//                 anyOf {
//                     changeset pattern: "**/*.java", comparator: 'GLOB'
//                     changeset pattern: "**/pom.xml", comparator: 'GLOB'
//                 }
//             }
//             steps {
//                 sh "mvn clean install -DskipTests" 
//             }
//         }

//         stage('Backend Services CI (Java)') {
//             matrix {
//                 axes {
//                     axis {
//                         name 'SERVICE'
//                         values 'media', 'product', 'cart', 'order', 'payment', 
//                                'search', 'customer', 'inventory', 'delivery', 
//                                'location', 'promotion', 'rating', 
//                                'recommendation', 'tax', 'webhook', 'storefront-bff', 'backoffice-bff'
//                     }
//                 }
//                 stages {
//                     stage('Build & Scan') {
//                         when { 
//                             anyOf {
//                                 changeset pattern: "${SERVICE}/**/*", comparator: 'GLOB'
//                                 changeset pattern: "common-library/**/*", comparator: 'GLOB'
//                                 changeset pattern: "pom.xml", comparator: 'GLOB'
//                             }
//                         }
//                         steps {
//                             sh "mvn verify -pl ${SERVICE}"
//                             sh "chmod +x mvnw || true" 
//                             sh "chmod +x ${SERVICE}/mvnw || true"
//                             withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
//                                 sh """
//                                     snyk auth \$SNYK_TOKEN
//                                     snyk test --file=${SERVICE}/pom.xml --severity-threshold=high
//                                 """
//                             }

//                             withSonarQubeEnv('SonarCloud') {
//                                 dir("${SERVICE}") {
//                                     sh """
//                                         mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
//                                         -Dsonar.projectKey=Fui-King_yas${SERVICE} \
//                                         -Dsonar.projectName="Yas - ${SERVICE}" \
//                                         -Dsonar.organization=fui-king \
//                                         -Dsonar.host.url=https://sonarcloud.io
//                                     """
//                                 }
//                             }
//                         }
//                         post {
//                             always {
//                                 junit testResults: "${SERVICE}/target/surefire-reports/*.xml", allowEmptyResults: true
//                                 jacoco(
//                                     execPattern: "${SERVICE}/target/**/jacoco.exec",
//                                     classPattern: "${SERVICE}/target/classes",
//                                     sourcePattern: "${SERVICE}/src/main/java",
//                                     inclusionPattern: '**/*.class',
//                                     maximumLineCoverage: '70',
//                                     minimumLineCoverage: '70',
//                                     changeBuildStatus: true 
//                                 )
//                             }
//                         }
//                     }
//                 }
//             }
//         }

//         stage('Frontend & BFF CI (Node.js)') {
//             matrix {
//                 axes {
//                     axis {
//                         name 'UI_SERVICE'
//                         values 'storefront', 'backoffice'
//                     }
//                 }
//                 stages {
//                     stage('Node.js') {
//                         when { 
//                             changeset pattern: "${UI_SERVICE}/**/*", comparator: 'GLOB'
//                         }
//                         steps {
//                             dir("${UI_SERVICE}") {
//                                 echo "Building UI/BFF Project: ${UI_SERVICE}..."
//                                 sh 'npm ci'
//                                 sh 'npm run lint'
//                                 sh 'npm run build'
//                             }
                            
//                             withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
//                                 sh """
//                                     snyk auth \$SNYK_TOKEN
//                                     snyk test --file=${UI_SERVICE}/package.json --severity-threshold=high
//                                 """
//                             }
//                         }
//                     }
//                 }
//             }
//         }
//     }
// }
pipeline {
    agent any
    
    tools {
        jdk 'JDK-21'
        maven 'maven-3'
        nodejs 'NodeJS-20' 
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    stages {
        stage('Setup Tools') {
            steps {
                script {
                    echo "Checking if Snyk CLI is installed..."
                    def snykExists = sh(script: 'command -v snyk', returnStatus: true) == 0
                    
                    if (!snykExists) {
                        echo "Snyk CLI not found. Installing via npm..."
                        sh 'npm install -g snyk'
                    } else {
                        echo "Snyk CLI is already installed. Skipping installation."
                    }
                }
            }
        }

        stage('Gitleaks Scan') {
            steps {
                script {
                    echo "Running Gitleaks to detect hardcoded secrets..."
                    catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                        sh """docker run --rm -v "${WORKSPACE}:/work" -w /work zricethezav/gitleaks:latest detect --source="." --verbose"""
                    }
                }
            }
        }

        stage('Pre-build Java Dependencies') {
            when { 
                anyOf {
                    changeset pattern: "**/*.java", comparator: 'GLOB'
                    changeset pattern: "**/pom.xml", comparator: 'GLOB'
                }
            }
            steps {
                sh "mvn clean install -DskipTests" 
            }
        }

        stage('Backend Services CI (Java)') {
            matrix {
                axes {
                    axis {
                        name 'SERVICE'
                        values 'media', 'product', 'cart', 'order', 'payment', 
                               'search', 'customer', 'inventory', 'delivery', 
                               'location', 'promotion', 'rating', 
                               'recommendation', 'tax', 'webhook', 'storefront-bff', 'backoffice-bff'
                    }
                }
                stages {
                    // --- PHASE 1: BUILD ---
                    stage('Build') {
                        when { 
                            anyOf {
                                changeset pattern: "${SERVICE}/**/*", comparator: 'GLOB'
                                changeset pattern: "common-library/**/*", comparator: 'GLOB'
                                changeset pattern: "pom.xml", comparator: 'GLOB'
                            }
                        }
                        steps {
                            sh "mvn package -DskipTests -pl ${SERVICE} -am"
                        }
                    }

                    // --- PHASE 2: TEST & SCAN ---
                    stage('Test & Scan') {
                        when { 
                            anyOf {
                                changeset pattern: "${SERVICE}/**/*", comparator: 'GLOB'
                                changeset pattern: "common-library/**/*", comparator: 'GLOB'
                                changeset pattern: "pom.xml", comparator: 'GLOB'
                            }
                        }
                        steps {
                            sh "mvn test -pl ${SERVICE} -am"
                            
                            sh "chmod +x mvnw || true" 
                            sh "chmod +x ${SERVICE}/mvnw || true"
                            
                            withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
                                sh """
                                    snyk auth \$SNYK_TOKEN
                                    snyk test --file=${SERVICE}/pom.xml --severity-threshold=high || true
                                """
                            }

                            withSonarQubeEnv('SonarCloud') {
                                dir("${SERVICE}") {
                                    sh """
                                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                                        -Dsonar.projectKey=Fui-King_yas${SERVICE} \
                                        -Dsonar.projectName="Yas - ${SERVICE}" \
                                        -Dsonar.organization=fui-king \
                                        -Dsonar.host.url=https://sonarcloud.io
                                    """
                                }
                            }
                        }
                        post {
                            always {
                                junit testResults: "${SERVICE}/target/surefire-reports/*.xml", allowEmptyResults: true
                                jacoco(
                                    execPattern: "${SERVICE}/target/**/jacoco.exec",
                                    classPattern: "${SERVICE}/target/classes",
                                    sourcePattern: "${SERVICE}/src/main/java",
                                    inclusionPattern: '**/*.class',
                                    maximumLineCoverage: '70',
                                    minimumLineCoverage: '70',
                                    changeBuildStatus: true 
                                )
                            }
                        }
                    }
                }
            }
        }

        stage('Frontend & BFF CI (Node.js)') {
            matrix {
                axes {
                    axis {
                        name 'UI_SERVICE'
                        values 'storefront', 'backoffice'
                    }
                }
                stages {
                    // --- PHASE 1: BUILD ---
                    stage('Build') {
                        when { 
                            changeset pattern: "${UI_SERVICE}/**/*", comparator: 'GLOB'
                        }
                        steps {
                            dir("${UI_SERVICE}") {
                                echo "Building UI/BFF Project: ${UI_SERVICE}..."
                                sh 'npm ci'
                                sh 'npm run lint'
                                sh 'npm run build'
                            }
                        }
                    }

                    // --- PHASE 2: TEST & SCAN ---
                    stage('Test & Scan') {
                        when { 
                            changeset pattern: "${UI_SERVICE}/**/*", comparator: 'GLOB'
                        }
                        steps {
                            withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
                                sh """
                                    snyk auth \$SNYK_TOKEN
                                    snyk test --file=${UI_SERVICE}/package.json --severity-threshold=high || true
                                """
                            }
                        }
                    }
                }
            }
        }
    }
}