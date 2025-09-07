pipeline {
    agent any

    environment {
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = '13.234.116.221'
        DOCUMENT_ROOT = '/var/www/vhosts/demo.cloudtechstudy.shop'
        VHOST_CONF = '/etc/apache2/sites-available/demo.cloudtechstudy.shop.conf'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/aniruddha-demo/Medilab.git'
            }
        }

        stage('Setup Apache Vhost on Remote Server') {
            steps {
                sshagent (credentials: ['your-ssh-credentials-id']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
                    sudo mkdir -p ${DOCUMENT_ROOT}
                    sudo chown -R ubuntu:ubuntu ${DOCUMENT_ROOT}

                    sudo bash -c 'cat > ${VHOST_CONF} << EOL
<VirtualHost *:80>
    ServerName demo.cloudtechstudy.shop
    DocumentRoot ${DOCUMENT_ROOT}

    <Directory ${DOCUMENT_ROOT}>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/demo.cloudtechstudy.shop_error.log
    CustomLog \${APACHE_LOG_DIR}/demo.cloudtechstudy.shop_access.log combined
</VirtualHost>
EOL'

                    sudo a2ensite demo.cloudtechstudy.shop.conf
                    sudo systemctl reload apache2
EOF
                    """
                }
            }
        }

        stage('Deploy Website Files with rsync') {
            steps {
                sshagent (credentials: ['your-ssh-credentials-id']) {
                    sh """
                    rsync -avz --exclude='.git/' --exclude='Jenkinsfile' -e "ssh -o StrictHostKeyChecking=no" ./ ${REMOTE_USER}@${REMOTE_HOST}:${DOCUMENT_ROOT}/
                    """
                }
            }
        }
    }
}

