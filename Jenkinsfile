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

// pipeline {
//     agent any
//     tools {
//         jdk 'JDK-21'
//         maven 'maven-3'
//         nodejs 'NodeJS-20'
//     }
//     environment {
//         // Thay đổi các biến này cho phù hợp với Jenkins của bạn
//         SONARQUBE_SERVER = 'SonarQube'
//         SONARQUBE_TOKEN = credentials('fuiking-sonar-token')
//         SNYK_TOKEN = credentials('fuiking-snyk-token')
//         GITLEAKS_CONFIG = 'gitleaks.toml'
//     }
//     options {
//         skipDefaultCheckout()
//         timestamps()
//     }
//     parameters {
//         string(name: 'SERVICE', defaultValue: '', description: 'Service to build/test (leave blank for auto-detect)')
//     }
//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }
//         stage('Detect Changed Service') {
//             steps {
//                 script {
//                     if (!params.SERVICE) {
//                         // So sánh commit hiện tại (HEAD) với commit trước đó (HEAD~1)
//                         def changed = sh(script: "git diff --name-only HEAD~1 HEAD | grep '/' | cut -d'/' -f1 | sort | uniq", returnStdout: true).trim()
                        
//                         if (changed) {
//                             env.SERVICE = changed.tokenize('\n')[0]
//                         } else {
//                             // Fix cứng thư mục mặc định nếu không nhận diện được, thay 'ten-thu-muc-service-cua-ban' cho đúng
//                             env.SERVICE = 'ten-thu-muc-service-cua-ban' 
//                         }
//                     }
//                     echo "Service to build/test: ${env.SERVICE}"
//                 }
//             }
//         }
//         stage('Gitleaks Scan') {
//             steps {
//                 script {
//                     echo "Bắt đầu quét Gitleaks qua Docker..."
//                     sh """
//                         docker run --rm \
//                         -v \$(pwd):/my-project \
//                         zricethezav/gitleaks:latest \
//                         detect --source="/my-project" --config="/my-project/${GITLEAKS_CONFIG}" --no-git || true
//                     """
//                 }
//             }
//         }
//         stage('Test & Coverage') {
//             steps {
//                 dir("${env.SERVICE}") {
//                     script {
//                         if (fileExists('pom.xml')) {
//                             sh 'mvn clean test'
//                             junit '**/target/surefire-reports/*.xml'
//                             sh 'mvn jacoco:report'
//                             // Đẩy coverage lên SonarQube
//                             withSonarQubeEnv("SonarCloud") {
//                                 sh """
//                                     mvn sonar:sonar \
//                                     -Dsonar.projectKey=Fui-King_yas \
//                                     -Dsonar.organization=fui-king \
//                                     -Dsonar.host.url=https://sonarcloud.io \
//                                     -Dsonar.token=${SONARQUBE_TOKEN}
//                                 """
//                             }
//                         } else if (fileExists('package.json')) {
//                             sh 'npm install'
//                             sh 'npm run test -- --coverage'
//                             junit 'coverage/junit.xml'
//                             // Đẩy coverage lên SonarQube
//                             withSonarQubeEnv("SonarCloud") {
//                                 sh """
//                                     mvn sonar:sonar \
//                                     -Dsonar.projectKey=Fui-King_yas \
//                                     -Dsonar.organization=fui-king \
//                                     -Dsonar.host.url=https://sonarcloud.io \
//                                     -Dsonar.token=${SONARQUBE_TOKEN}
//                                 """
//                             }
//                         }
//                     }
//                 }
//             }
//             post {
//                 always {
//                     // Đảm bảo upload kết quả test
//                     junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml,**/coverage/junit.xml'
//                 }
//             }
//         }
//         stage('Check Coverage') {
//             steps {
//                 script {
//                     // Đọc file coverage report và kiểm tra threshold
//                     def coverage = 0
//                     if (fileExists("${env.SERVICE}/target/site/jacoco/index.html")) {
//                         coverage = sh(script: "grep -oP 'Total.*?\\K[0-9]+(?=%)' ${env.SERVICE}/target/site/jacoco/index.html | head -1", returnStdout: true).trim()
//                     } else if (fileExists("${env.SERVICE}/coverage/coverage-summary.json")) {
//                         coverage = sh(script: "jq '.total.lines.pct' ${env.SERVICE}/coverage/coverage-summary.json", returnStdout: true).trim()
//                     }
//                     echo "Coverage: ${coverage}%"
//                     if (coverage.toInteger() < 0) { // Chỉnh lại sau này
//                         error "Test coverage is below 20%!"
//                     }
//                 }
//             }
//         }
//         stage('Snyk Scan') {
//             steps {
//                 dir("${env.SERVICE}") {
//                     script {
//                         if (fileExists('pom.xml')) {
//                             sh "snyk test --all-projects --org=your-org --token=${SNYK_TOKEN} || true"
//                         } else if (fileExists('package.json')) {
//                             sh "snyk test --all-projects --org=your-org --token=${SNYK_TOKEN} || true"
//                         }
//                     }
//                 }
//             }
//         }
//         stage('Build') {
//             steps {
//                 dir("${env.SERVICE}") {
//                     script {
//                         if (fileExists('pom.xml')) {
//                             sh 'mvn clean package -DskipTests'
//                         } else if (fileExists('package.json')) {
//                             sh 'npm run build'
//                         }
//                     }
//                 }
//             }
//         }
//     }
//     post {
//         always {
//             cleanWs()
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

    environment {
        // Cấu hình ID Tổ chức Snyk của bạn (Thay bằng ID thật trên Snyk)
        SNYK_ORG = 'your-org-id' 
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    stages {
        // ==========================================
        // STAGE 1: GLOBAL SECURITY SCAN (GITLEAKS)
        // ==========================================
        stage('Global Security Scan') {
            steps {
                script {
                    echo "Running Gitleaks to detect hardcoded secrets..."
                    // Nếu phát hiện secret lộ, stage sẽ báo vàng (UNSTABLE) nhưng vẫn chạy tiếp các service
                    catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                        sh """
                            docker run --rm -v \$(pwd):/my-project zricethezav/gitleaks:latest detect --source="/my-project" --no-git --verbose
                        """
                    }
                }
            }
        }

        // ==========================================
        // STAGE 2: BACKEND SERVICES (JAVA / SPRING BOOT)
        // ==========================================
        stage('Backend Services CI (Java)') {
            matrix {
                axes {
                    axis {
                        name 'SERVICE'
                        values 'media', 'product', 'cart', 'order', 'payment', 'search', 'customer', 'inventory', 'delivery', 'location', 'promotion', 'rating', 'recommendation', 'tax', 'webhook'
                    }
                }
                stages {
                    // --- PHASE 1: TEST & SCAN ---
                    stage('Test Phase') {
                        when { 
                            anyOf {
                                changeset pattern: "${SERVICE}/**/*", comparator: 'GLOB'
                                changeset pattern: "common-library/**/*", comparator: 'GLOB'
                                changeset pattern: "pom.xml", comparator: 'GLOB'
                            }
                        }
                        steps {
                            dir("${SERVICE}") {
                                echo "=== [TEST] Đang xử lý Backend Service: ${SERVICE} ==="
                                
                                // 1. Chạy Unit Test & tạo report Jacoco
                                sh 'mvn clean test jacoco:report -DfailIfNoTests=false'
                                
                                // 2. SonarQube Scan
                                withSonarQubeEnv('SonarCloud') {
                                    sh """
                                        mvn sonar:sonar \
                                        -Dsonar.projectKey=Fui-King_yas_${SERVICE} \
                                        -Dsonar.organization=fui-king \
                                        -Dsonar.moduleKey=${SERVICE} \
                                        -Dsonar.projectName="YAS - ${SERVICE}" \
                                        -Dsonar.host.url=https://sonarcloud.io \
                                        -Dsonar.branch.name=main
                                    """
                                }

                                // 3. Snyk Scan (Bắt buộc fail nếu có lỗ hổng High/Critical)
                                withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
                                    sh """
                                        snyk auth \$SNYK_TOKEN
                                        snyk test --file=pom.xml --severity-threshold=high --org=${SNYK_ORG}
                                    """
                                }
                            }
                        }
                        post {
                            always {
                                // Upload Test Results (Hiển thị tab Test)
                                junit testResults: "${SERVICE}/target/surefire-reports/*.xml", allowEmptyResults: true
                                
                                // Upload Coverage & Chặn nếu < 70% (Hiển thị tab Coverage)
                                jacoco(
                                    execPattern: "${SERVICE}/target/**/jacoco.exec",
                                    classPattern: "${SERVICE}/target/classes",
                                    sourcePattern: "${SERVICE}/src/main/java",
                                    inclusionPattern: '**/*.class',
                                    maximumLineCoverage: '70',
                                    minimumLineCoverage: '70',
                                    changeBuildStatus: true // Sẽ đánh fail pipeline nếu không đủ 70%
                                )
                            }
                        }
                    }

                    // --- PHASE 2: BUILD ---
                    stage('Build Phase') {
                        when { 
                            anyOf {
                                changeset pattern: "${SERVICE}/**/*", comparator: 'GLOB'
                                changeset pattern: "common-library/**/*", comparator: 'GLOB'
                                changeset pattern: "pom.xml", comparator: 'GLOB'
                            }
                        }
                        steps {
                            dir("${SERVICE}") {
                                echo "=== [BUILD] Đang đóng gói Backend Service: ${SERVICE} ==="
                                // Đóng gói thành .jar (bỏ qua test vì đã chạy ở trên)
                                sh 'mvn package -DskipTests'
                            }
                        }
                    }
                }
            }
        }

        // ==========================================
        // STAGE 3: FRONTEND & BFF SERVICES (NODE.JS)
        // ==========================================
        stage('Frontend & BFF CI (Node.js)') {
            matrix {
                axes {
                    axis {
                        name 'UI_SERVICE'
                        values 'storefront', 'backoffice', 'storefront-bff', 'backoffice-bff'
                    }
                }
                stages {
                    // --- PHASE 1: TEST & SCAN ---
                    stage('Test Phase') {
                        when { 
                            changeset pattern: "${UI_SERVICE}/**/*", comparator: 'GLOB' 
                        }
                        steps {
                            dir("${UI_SERVICE}") {
                                echo "=== [TEST] Đang xử lý UI/BFF Service: ${UI_SERVICE} ==="
                                
                                // 1. Cài đặt thư viện & Chạy Test
                                sh 'npm ci'
                                sh 'npm run test -- --coverage'
                                
                                // 2. Kiểm tra Coverage > 70% thủ công qua file JSON
                                script {
                                    if (fileExists("coverage/coverage-summary.json")) {
                                        // Đòi hỏi Jenkins server phải cài gói 'jq'
                                        def coverage = sh(script: "jq '.total.lines.pct' coverage/coverage-summary.json", returnStdout: true).trim()
                                        echo "Node.js Service ${UI_SERVICE} Coverage: ${coverage}%"
                                        
                                        if (coverage.toDouble() < 70.0) {
                                            error "Test coverage của ${UI_SERVICE} là ${coverage}%, không đạt mức tối thiểu 70%!"
                                        }
                                    } else {
                                        echo "Warning: Không tìm thấy file coverage-summary.json"
                                    }
                                }

                                // 3. SonarQube Scan
                                withSonarQubeEnv('SonarCloud') {
                                    sh """
                                        sonar-scanner \
                                        -Dsonar.projectKey=Fui-King_yas_${UI_SERVICE} \
                                        -Dsonar.organization=fui-king \
                                        -Dsonar.sources=. \
                                        -Dsonar.host.url=https://sonarcloud.io \
                                        -Dsonar.branch.name=main
                                    """
                                }

                                // 4. Snyk Scan
                                withCredentials([string(credentialsId: 'fuiking-snyk-token', variable: 'SNYK_TOKEN')]) {
                                    sh """
                                        snyk auth \$SNYK_TOKEN
                                        snyk test --file=package.json --severity-threshold=high --org=${SNYK_ORG}
                                    """
                                }
                            }
                        }
                        post {
                            always {
                                // Upload Test Results
                                junit testResults: "${UI_SERVICE}/coverage/junit.xml", allowEmptyResults: true
                                
                                // Nén thư mục coverage thành Artifact để download làm bằng chứng
                                archiveArtifacts artifacts: "${UI_SERVICE}/coverage/**", allowEmptyArchive: true
                            }
                        }
                    }

                    // --- PHASE 2: BUILD ---
                    stage('Build Phase') {
                        when { 
                            changeset pattern: "${UI_SERVICE}/**/*", comparator: 'GLOB' 
                        }
                        steps {
                            dir("${UI_SERVICE}") {
                                echo "=== [BUILD] Đang build UI/BFF Service: ${UI_SERVICE} ==="
                                sh 'npm run build'
                            }
                        }
                    }
                }
            }
        }
    }

    // ==========================================
    // CLEANUP & NOTIFICATION
    // ==========================================
    post {
        always {
            cleanWs() // Dọn dẹp Workspace sau khi chạy xong để giải phóng bộ nhớ Jenkins
        }
        success {
            echo "✅ Pipeline Passed! Code an toàn và đạt chuẩn độ phủ."
        }
        failure {
            echo "❌ Pipeline Failed! Kiểm tra lại Coverage (< 70%), Unit Test lỗi, hoặc dính lỗi bảo mật từ Snyk."
        }
    }
}