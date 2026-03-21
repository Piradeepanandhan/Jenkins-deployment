pipeline {
    agent any

    environment {
        APP_NAME = "node-app"
        APP_PORT = "3000"
    }

    stages {

        stage('Clone Repo') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run App with PM2') {
            steps {
                sh '''
                pm2 describe $APP_NAME > /dev/null 2>&1

                if [ $? -ne 0 ]; then
                    echo "Starting new PM2 app..."
                    pm2 start app.js --name $APP_NAME
                else
                    echo "Restarting existing PM2 app..."
                    pm2 restart $APP_NAME
                fi

                pm2 save
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
                    echo "Nginx already installed. Skipping..."
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
                else
                    echo "Nginx already configured. Skipping..."
                fi
                '''
            }
        }
    }
}