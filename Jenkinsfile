pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run Python Tests') {
            steps {
                sh 'python3 -m unittest discover -v'
            }
        }
    }

    post {
        success {
            echo 'BUILD SUCCESSFUL - All Python tests passed!'
        }

        failure {
            echo 'BUILD FAILED - Check the test results.'
        }

        always {
            sh '''
                set +x
                python3 - <<'PY'
import json
import os
import urllib.request

config = {}
with open("/var/jenkins_home/webex.env") as f:
    for line in f:
        line = line.strip()
        if "=" in line and not line.startswith("#"):
            key, value = line.split("=", 1)
            config[key] = value.strip().strip('"').strip("'")

status = os.environ.get("currentBuild", "")
result = os.environ.get("BUILD_RESULT", "Completed")

message = "Jenkins CI/CD build finished. Check Jenkins for the result."

data = json.dumps({
    "roomId": config["WEBEX_ROOM_ID"],
    "text": message
}).encode()

request = urllib.request.Request(
    "https://webexapis.com/v1/messages",
    data=data,
    headers={
        "Authorization": "Bearer " + config["WEBEX_BOT_TOKEN"],
        "Content-Type": "application/json"
    },
    method="POST"
)

with urllib.request.urlopen(request, timeout=15) as response:
    print("Webex notification HTTP status:", response.status)
PY
            '''
        }
    }
}
