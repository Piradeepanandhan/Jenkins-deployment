pipeline {
    agent any

    environment {
        APP_NAME = "node-app"
        APP_PORT = "3000"
        PM2_HOME = "/var/lib/jenkins/.pm2"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                echo "Installing dependencies..."
                npm install
                '''
            }
        }

        stage('Run App with PM2') {
            steps {
                sh '''
                export PM2_HOME=$PM2_HOME

                echo "Using PM2_HOME=$PM2_HOME"

                # Install PM2 if not present
                if ! command -v pm2 > /dev/null; then
                    echo "Installing PM2..."
                    sudo npm install -g pm2
                fi

                # Restore previous processes (if any)
                pm2 resurrect || true

                # Start or restart app
                if pm2 describe $APP_NAME > /dev/null 2>&1; then
                    echo "Restarting existing app..."
                    pm2 restart $APP_NAME
                else
                    echo "Starting new app..."
                    pm2 start app.js --name $APP_NAME
                fi

                # Save process list
                pm2 save
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Waiting for app..."
                sleep 5

                curl -f http://localhost:$APP_PORT || exit 1

                echo "App is running successfully!"
                '''
            }
        }

        stage('Install Nginx (If Not Installed)') {
            steps {
                sh '''
                if ! command -v nginx > /dev/null; then
                    echo "Installing Nginx..."
                    sudo apt update -y
                    sudo apt install -y nginx
                else
                    echo "Nginx already installed → Skipping"
                fi
                '''
            }
        }

        stage('Configure Nginx (Only First Time)') {
            steps {
                sh '''
                if [ ! -f /etc/nginx/sites-available/node-app ]; then
                    echo "Configuring Nginx..."

                    sudo bash -c 'cat > /etc/nginx/sites-available/node-app <<EOF
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \\$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \\$host;
        proxy_cache_bypass \\$http_upgrade;
    }
}
EOF'

                    sudo ln -s /etc/nginx/sites-available/node-app /etc/nginx/sites-enabled/
                    sudo rm -f /etc/nginx/sites-enabled/default

                    sudo nginx -t
                    sudo systemctl restart nginx

                    echo "Nginx configured!"
                else
                    echo "Nginx already configured → Skipping"
                fi
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
}
