post {
    always {
        archiveArtifacts artifacts: '**/*.html', allowEmptyArchive: true

        publishHTML([
            reportDir: 'test-report',
            reportFiles: 'index.html',
            reportName: 'TestNG HTML Test Report',
            keepAll: true,
            alwaysLinkToLastBuild: true,
            allowMissing: true
        ])
    }

    success {
        echo "Build Successful"
    }

    failure {
        echo "Build Failed"
    }
}
