pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Installing Python dependencies...'
                dir('python-flask-app') {
                    sh '''
                        python3 -m venv .venv
                        .venv/bin/pip install --upgrade pip
                        .venv/bin/pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                dir('python-flask-app') {
                    sh '.venv/bin/python -m unittest discover -s .'
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                    mkdir -p "${WORKSPACE}/python-app-deploy"
                    cp "${WORKSPACE}/python-flask-app/app.py" "${WORKSPACE}/python-app-deploy/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application...'
                sh '''
                    nohup "${WORKSPACE}/python-flask-app/.venv/bin/python" \
                    "${WORKSPACE}/python-app-deploy/app.py" \
                    > "${WORKSPACE}/python-app-deploy/app.log" 2>&1 &
                    echo $! > "${WORKSPACE}/python-app-deploy/app.pid"
                    sleep 3
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing deployed application...'
                sh '''
                    "${WORKSPACE}/python-flask-app/.venv/bin/python" - <<'PY'
import time
import urllib.request

url = "http://127.0.0.1:5000/"
for attempt in range(20):
    try:
        response = urllib.request.urlopen(url, timeout=2)
        body = response.read().decode()
        print(body)
        assert response.status == 200
        assert body == "Hello, Jenkins Multi-Stage Pipeline!"
        print("Application test passed!")
        break
    except Exception:
        if attempt == 19:
            raise
        time.sleep(1)
PY
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
        always {
            sh '''
                if [ -f "${WORKSPACE}/python-app-deploy/app.pid" ]; then
                    kill "$(cat "${WORKSPACE}/python-app-deploy/app.pid")" 2>/dev/null || true
                fi
            '''
        }
    }
}