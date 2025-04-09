# Complete PHP Setup Guide for Ubuntu with Visual Studio Code

This guide covers the installation and configuration of PHP on Ubuntu, including Apache, MySQL, and VS Code integration.

## 1. Install the LAMP Stack

### Update System Packages
```bash
sudo apt update
sudo apt upgrade -y
```

### Install Apache Web Server
```bash
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
```

### Install MySQL
```bash
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo systemctl enable mysql
```

### Secure MySQL Installation
```bash
sudo mysql_secure_installation
```
Follow the prompts to:
- Set a root password
- Remove anonymous users
- Disallow root login remotely
- Remove test database
- Reload privilege tables

### Install PHP (Version 8.4)
```bash
# Add the repository for latest PHP versions
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP 8.4 with common extensions
sudo apt install php8.4 libapache2-mod-php8.4 php8.4-mysql php8.4-common php8.4-cli -y

# Install additional useful PHP extensions
sudo apt install php8.4-curl php8.4-xml php8.4-zip php8.4-mbstring php8.4-gd php8.4-intl php8.4-opcache -y
```

### Enable PHP in Apache
```bash
sudo a2enmod php8.4
sudo systemctl restart apache2
```

## 2. Test PHP Installation

### Create a PHP Info File
```bash
sudo nano /var/www/html/phpinfo.php
```

Add the following content:
```php
<?php
phpinfo();
?>
```

Save and exit, then set proper permissions:
```bash
sudo chown www-data:www-data /var/www/html/phpinfo.php
sudo chmod 644 /var/www/html/phpinfo.php
```

### Test in Browser
Visit `http://localhost/phpinfo.php` in your browser to verify that PHP is working correctly.

## 3. Set Up Visual Studio Code for PHP Development

### Install VS Code
```bash
# Download and install the .deb package
sudo apt install software-properties-common apt-transport-https wget -y
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
sudo apt update
sudo apt install code -y
```

### Install Required PHP Extensions for VS Code
Open VS Code and install these extensions:
1. PHP IntelliSense by Felix Becker
2. PHP Debug by Xdebug
3. PHP Formatter
4. PHP Namespace Resolver
5. PHP DocBlocker

To install extensions via command line:
```bash
code --install-extension felixfbecker.php-intellisense
code --install-extension xdebug.php-debug
code --install-extension bmewburn.vscode-intelephense-client
code --install-extension MehediDracula.php-namespace-resolver
code --install-extension neilbrayfield.php-docblocker
php -S localhost:8000

```

### Configure PHP in VS Code
1. Open VS Code settings (File > Preferences > Settings or `Ctrl+,`)
2. Search for "php.validate.executablePath"
3. Set it to: `/usr/bin/php8.4`

Alternatively, edit settings.json directly:
```bash
mkdir -p ~/.config/Code/User/
cat > ~/.config/Code/User/settings.json << EOL
{
    "php.validate.executablePath": "/usr/bin/php8.4",
    "php.suggest.basic": true
}
EOL
```

## 4. Create a PHP Project Structure

### Create a Project Directory
```bash
# Create a projects directory in your home folder
mkdir -p ~/php-projects/my-first-project
cd ~/php-projects/my-first-project

# Create a basic PHP file
cat > index.php << EOL
<!DOCTYPE html>
<html>
<head>
    <title>My PHP Project</title>
</head>
<body>
    <h1>Welcome to PHP Development</h1>
    
    <?php
        echo "<p>Today is " . date("Y-m-d H:i:s") . "</p>";
        echo "<p>PHP Version: " . phpversion() . "</p>";
    ?>
</body>
</html>
EOL

# Link to web server directory (optional)
sudo ln -s ~/php-projects/my-first-project /var/www/html/my-project
sudo chown -R $USER:$USER ~/php-projects
sudo chown -R www-data:www-data /var/www/html/my-project
```

### Open Your Project in VS Code
```bash
code ~/php-projects/my-first-project
```

## 5. Set Up Apache Virtual Hosts (Optional)

### Create a Virtual Host Configuration
```bash
sudo nano /etc/apache2/sites-available/my-project.conf
```

Add the following content:
```apache
<VirtualHost *:80>
    ServerName my-project.local
    DocumentRoot /var/www/html/my-project
    
    <Directory /var/www/html/my-project>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/my-project_error.log
    CustomLog ${APACHE_LOG_DIR}/my-project_access.log combined
</VirtualHost>
```

### Enable the Virtual Host
```bash
sudo a2ensite my-project.conf
sudo systemctl reload apache2
```

### Add Local Domain to Hosts File
```bash
sudo nano /etc/hosts
```

Add this line:
```
127.0.0.1   my-project.local
```

## 6. Install and Configure Xdebug (Optional)

### Install Xdebug Extension
```bash
sudo apt install php8.4-xdebug -y
```

### Configure Xdebug
```bash
sudo nano /etc/php/8.4/mods-available/xdebug.ini
```

Add the following content:
```ini
zend_extension=xdebug.so
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_port=9003
xdebug.client_host=127.0.0.1
xdebug.log=/var/log/xdebug.log
```

### Restart Apache
```bash
sudo systemctl restart apache2
```

### Configure VS Code for Debugging
Create a `.vscode/launch.json` file in your project:

```bash
mkdir -p ~/php-projects/my-first-project/.vscode
cat > ~/php-projects/my-first-project/.vscode/launch.json << EOL
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003
        }
    ]
}
EOL
```

## 7. Basic PHP Workflow with VS Code

1. Create PHP files in your project directory
2. Edit them with VS Code's PHP support (syntax highlighting, autocompletion)
3. Test your code by accessing http://localhost/my-project/ in your browser
4. Use VS Code's integrated terminal (`Ctrl+``) to run PHP scripts directly:
   ```bash
   php -f yourscript.php
   ```
5. Debug your PHP code using the VS Code debugging tools (`F5`)

## 8. Common PHP Extensions for Development

Install additional PHP extensions as needed:
```bash
# For image processing
sudo apt install php8.4-gd -y

# For internationalization
sudo apt install php8.4-intl -y

# For JSON support
sudo apt install php8.4-json -y

# For SQLite database
sudo apt install php8.4-sqlite3 -y

# For SOAP web services
sudo apt install php8.4-soap -y
```

## 9. Composer Installation (PHP Package Manager)

```bash
# Download Composer installer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Verify installer
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e21205b207c3ff031906575712edab6f13eb0b361f2085f1f1237b7126d785e826a450292b6cfd1d64d92e6563bbde02') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# Install Composer globally
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# Clean up
php -r "unlink('composer-setup.php');"
```

## 10. Troubleshooting

### Apache Issues
- Check Apache status: `sudo systemctl status apache2`
- Check error logs: `sudo tail -f /var/log/apache2/error.log`
- Test Apache config: `sudo apache2ctl configtest`

### PHP Issues
- Verify PHP version: `php -v`
- Check PHP modules: `php -m`
- Check PHP configuration: `php --ini`

### Permissions Issues
- Fix web directory permissions: `sudo chown -R www-data:www-data /var/www/html`
- Check file permissions: `ls -la /var/www/html/`

### VS Code Issues
- Reload window: Press `Ctrl+Shift+P`, type "Reload Window" and press Enter
- Check extensions: Press `Ctrl+Shift+X` to view installed extensions
- Reset VS Code user settings: Delete or rename `~/.config/Code/User/settings.json`

## Additional Resources

- [PHP Documentation](https://www.php.net/docs.php)
- [VS Code PHP Development](https://code.visualstudio.com/docs/languages/php)
- [Composer Documentation](https://getcomposer.org/doc/)
- [PHP The Right Way](https://phptherightway.com/)
