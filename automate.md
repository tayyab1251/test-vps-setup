Local → GitHub → Webhook → VPS → Deploy to /var/www/your-project

1. create a repo in github and connect that repo to your local setup(laptop)


2. Create Project Directory on VPS

    mkdir /var/www/html/test-vps-setup

    cd /var/www/html/test-vps-setup

    git clone https://github.com/USERNAME/REPO.git .

    Fix ownership:

    chown -R www-data:www-data /var/www/test-vps-setup
    chmod -R 755 /var/www/test-vps-setup


3. Create Secret Token for later use to while accessing accessing the file

    openssl rand -hex 32
    for example we get  : 5c6039622c4a7784af2e30580ac59c5bd6f7596d67c083c34e418e6d8367c71d


4. Create deploy.php on VPS

    Path: /var/www/html/test-vps-setup/deploy.php

        <!-- Minimum Code for deploy.php code -->

        <?php
        // Simple token check
        $expected = 'f870226318a250248f1a840d665f3376c623b500bfb6355d9d87301fa0b85998';

        // Accept token either as query param or from header (flexible)
        $token = $_GET['token'] ?? ($_SERVER['HTTP_X_DEPLOY_TOKEN'] ?? null);

        if (!$token || !hash_equals($expected, $token)) {
            http_response_code(403);
            echo "Forbidden\n";
            exit;
        }

        // Optional: ensure request method is POST
        if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
            http_response_code(405);
            echo "Method Not Allowed\n";
            exit;
        }

        // Run deploy
        $output = shell_exec('bash /var/www/html/test-vps-setup/deploy.sh 2>&1');
        echo "<pre>$output</pre>";




5. Test Deploy Script Manually with cURL

    From your local machine:

    curl "http://YOUR_SERVER_IP/test-vps-setup/deploy.php?token=SECRET_TOKEN"

    Expected output:

    Pulling latest changes...
    Updating xxxxx..yyyyy
    Fast-forward
    Setting permissions...
    Done.


6. Setup GitHub Webhook

GitHub Repo → Settings → Webhooks → Add Webhook

Payload URL: http://YOUR_SERVER_IP/test-vps-setup/deploy.php?token=SECRET_TOKEN
Secret: 'PASTE HERE TOKEN THAT WE CREATED ABOVE'
Content Type: application/json
Event: "Just the push event"


7. Deployment Workflow (Final)

Now add some changes in your local repo that we are trying to connect and push, 

what ever the updates we pushed will be available on the VPS


8. Optional: Add deploy.sh

Create file: /var/www/html/test-vps-setup/deploy.sh

Content:

    #!/bin/bash
    cd /var/www/test-vps-setup
    git pull # here we can set owhich origin to pull like : git pull origin main
    chown -R www-data:www-data /var/www/html/test-vps-setup
    chmod -R 755 /var/www/html/test-vps-setup


9. Make it executable:

chmod +x deploy.sh


Call it from PHP if needed:

echo shell_exec("bash deploy.sh 2>&1");

