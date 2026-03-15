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
//                     echo "Checking Snyk CLI..."
//                     def snykExists = sh(script: 'command -v snyk', returnStatus: true) == 0
//                     if (!snykExists) { sh 'npm install -g snyk' }
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
//                     stage('Build') {
//                         when { 
//                             anyOf {
//                                 changeset pattern: "${SERVICE}/**/*", comparator: 'GLOB'
//                                 changeset pattern: "common-library/**/*", comparator: 'GLOB'
//                                 changeset pattern: "pom.xml", comparator: 'GLOB'
//                             }
//                         }
//                         steps {
//                             // Bỏ clean để tránh lỗi file lock khi chạy song song
//                             sh "mvn package -DskipTests -pl ${SERVICE} -am"
//                         }
//                     }

//                     stage('Test & Scan') {
//                         when { 
//                             anyOf {
//                                 changeset pattern: "${SERVICE}/**/*", comparator: 'GLOB'
//                                 changeset pattern: "common-library/**/*", comparator: 'GLOB'
//                                 changeset pattern: "pom.xml", comparator: 'GLOB'
//                             }
//                         }
//                         steps {
//                             sh "mvn test -pl ${SERVICE} -am"
                            
//                             sh "chmod +x mvnw || true" 
//                             sh "chmod +x ${SERVICE}/mvnw || true"
                            
//                             // Snyk Scan
//                             withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
//                                 sh """
//                                     snyk auth \$SNYK_TOKEN
//                                     snyk test --file=${SERVICE}/pom.xml --severity-threshold=high || true
//                                 """
//                             }

//                             // SonarQube Scan - Chạy trong thư mục service để tránh lỗi Reactor
//                             withSonarQubeEnv('SonarCloud') {
//                                 dir("${SERVICE}") {
//                                     // Sử dụng catchError để Stage chỉ bị UNSTABLE (Vàng) nếu Sonar lỗi, không làm chết Pipeline
//                                     catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
//                                         sh """
//                                             mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
//                                             -Dsonar.projectKey=Fui-King_yas \
//                                             -Dsonar.organization=fui-king \
//                                             -Dsonar.moduleKey=${SERVICE} \
//                                             -Dsonar.projectName="Yas - ${SERVICE}" \
//                                             -Dsonar.host.url=https://sonarcloud.io \
//                                             -Dsonar.branch.name=main
//                                         """
//                                     }
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
//                     stage('Build') {
//                         when { changeset pattern: "${UI_SERVICE}/**/*", comparator: 'GLOB' }
//                         steps {
//                             dir("${UI_SERVICE}") {
//                                 sh 'npm ci'
//                                 sh 'npm run lint'
//                                 sh 'npm run build'
//                             }
//                         }
//                     }
//                     stage('Test & Scan') {
//                         when { changeset pattern: "${UI_SERVICE}/**/*", comparator: 'GLOB' }
//                         steps {
//                             withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
//                                 sh """
//                                     snyk auth \$SNYK_TOKEN
//                                     snyk test --file=${UI_SERVICE}/package.json --severity-threshold=high || true
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
    environment {
        // Thay đổi các biến này cho phù hợp với Jenkins của bạn
        SONARQUBE_SERVER = 'SonarQube'
        SONARQUBE_TOKEN = credentials('fuiking-sonar-token')
        SNYK_TOKEN = credentials('fuiking-snyk-token')
        GITLEAKS_CONFIG = 'gitleaks.toml'
    }
    options {
        skipDefaultCheckout()
        timestamps()
    }
    parameters {
        string(name: 'SERVICE', defaultValue: '', description: 'Service to build/test (leave blank for auto-detect)')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Detect Changed Service') {
            steps {
                script {
                    // Tự động xác định service thay đổi nếu chưa truyền vào
                    if (!params.SERVICE) {
                        def changed = sh(script: "git diff --name-only origin/main...HEAD | grep '/' | cut -d'/' -f1 | sort | uniq", returnStdout: true).trim()
                        env.SERVICE = changed.tokenize('\n')[0] // Lấy service đầu tiên thay đổi
                    }
                    echo "Service to build/test: ${env.SERVICE}"
                }
            }
        }
        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source=. --config=$GITLEAKS_CONFIG --no-git'
            }
        }
        stage('Test & Coverage') {
            steps {
                dir("${env.SERVICE}") {
                    script {
                        if (fileExists('pom.xml')) {
                            sh 'mvn clean test'
                            junit '**/target/surefire-reports/*.xml'
                            sh 'mvn jacoco:report'
                            // Đẩy coverage lên SonarQube
                            withSonarQubeEnv("${SONARQUBE_SERVER}") {
                                sh "mvn sonar:sonar -Dsonar.login=${SONARQUBE_TOKEN}"
                            }
                        } else if (fileExists('package.json')) {
                            sh 'npm install'
                            sh 'npm run test -- --coverage'
                            junit 'coverage/junit.xml'
                            // Đẩy coverage lên SonarQube
                            withSonarQubeEnv("${SONARQUBE_SERVER}") {
                                sh "npx sonar-scanner -Dsonar.login=${SONARQUBE_TOKEN}"
                            }
                        }
                    }
                }
            }
            post {
                always {
                    // Đảm bảo upload kết quả test
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml,**/coverage/junit.xml'
                }
            }
        }
        stage('Check Coverage') {
            steps {
                script {
                    // Đọc file coverage report và kiểm tra threshold
                    def coverage = 0
                    if (fileExists("${env.SERVICE}/target/site/jacoco/index.html")) {
                        coverage = sh(script: "grep -oP 'Total.*?\\K[0-9]+(?=%)' ${env.SERVICE}/target/site/jacoco/index.html | head -1", returnStdout: true).trim()
                    } else if (fileExists("${env.SERVICE}/coverage/coverage-summary.json")) {
                        coverage = sh(script: "jq '.total.lines.pct' ${env.SERVICE}/coverage/coverage-summary.json", returnStdout: true).trim()
                    }
                    echo "Coverage: ${coverage}%"
                    if (coverage.toInteger() < 70) {
                        error "Test coverage is below 70%!"
                    }
                }
            }
        }
        stage('Snyk Scan') {
            steps {
                dir("${env.SERVICE}") {
                    script {
                        if (fileExists('pom.xml')) {
                            sh "snyk test --all-projects --org=your-org --token=${SNYK_TOKEN} || true"
                        } else if (fileExists('package.json')) {
                            sh "snyk test --all-projects --org=your-org --token=${SNYK_TOKEN} || true"
                        }
                    }
                }
            }
        }
        stage('Build') {
            steps {
                dir("${env.SERVICE}") {
                    script {
                        if (fileExists('pom.xml')) {
                            sh 'mvn clean package -DskipTests'
                        } else if (fileExists('package.json')) {
                            sh 'npm run build'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}