# Setting Up a Local WordPress Site with Apache, MySQL, and Git

This guide explains how to install Apache, clone a WordPress site from a Git repository, install MySQL, configure the site to run locally, and import a database backup.

## Prerequisites
- A Linux system (e.g., Ubuntu 22.04; notes for macOS/Windows included).
- A Git repository with your WordPress code (e.g., `https://github.com/yourusername/my-wp-site.git`).
- A database backup file (e.g., `my-wp-site.sql`).
- Basic terminal familiarity.

## Step 1: Install Apache
Apache is a web server to host your WordPress site locally.

### On Ubuntu
1. Update package lists:
   ```bash
   sudo apt update
   ```
2. Install Apache:
   ```bash
   sudo apt install apache2
   ```
3. Start Apache and enable it to run on boot:
   ```bash
   sudo systemctl start apache2
   sudo systemctl enable apache2
   ```
4. Verify Apache is running:
   - Open a browser and navigate to `http://localhost`. You should see the Apache default page.
   - Or run: `sudo systemctl status apache2`.

### On macOS
1. Use Homebrew to install Apache:
   ```bash
   brew install httpd
   ```
2. Start Apache:
   ```bash
   brew services start httpd
   ```
3. Verify by visiting `http://localhost:8080` (macOS Apache uses port 8080 by default).

### On Windows
1. Download and install XAMPP (includes Apache, MySQL, and PHP) from https://www.apachefriends.org/.
2. Start the XAMPP Control Panel and enable the Apache module.
3. Verify by visiting `http://localhost` in a browser.

## Step 2: Install MySQL
MySQL is the database server for WordPress.

### On Ubuntu
1. Install MySQL:
   ```bash
   sudo apt install mysql-server
   ```
2. Start MySQL and enable it:
   ```bash
   sudo systemctl start mysql
   sudo systemctl enable mysql
   ```
3. Secure the MySQL installation:
   ```bash
   sudo mysql_secure_installation
   ```
   - Follow prompts to set a root password and secure settings.
4. Log in to MySQL to verify:
   ```bash
   mysql -u root -p
   ```
   - Enter the root password and confirm you can access the MySQL prompt.

### On macOS
1. Install MySQL via Homebrew:
   ```bash
   brew install mysql
   ```
2. Start MySQL:
   ```bash
   brew services start mysql
   ```
3. Set a root password:
   ```bash
   mysqladmin -u root password 'yourpassword'
   ```
4. Verify by logging in:
   ```bash
   mysql -u root -p
   ```

### On Windows (with XAMPP)
- MySQL (MariaDB) is included with XAMPP. Start it from the XAMPP Control Panel.
- Access phpMyAdmin at `http://localhost/phpmyadmin` to manage databases.

## Step 3: Install PHP
WordPress requires PHP to process dynamic content.

### On Ubuntu
1. Install PHP and required modules:
   ```bash
   sudo apt install php php-mysql libapache2-mod-php php-gd php-mbstring php-xml php-curl
   ```
2. Verify PHP version:
   ```bash
   php -v
   ```
3. Enable PHP in Apache:
   - Ensure the PHP module is loaded (usually automatic after installation).

### On macOS
1. Install PHP via Homebrew:
   ```bash
   brew install php
   ```
2. Link PHP to Apache:
   - Edit Apache config (`/usr/local/etc/httpd/httpd.conf`) to load the PHP module:
     ```apache
     LoadModule php_module /usr/local/opt/php/lib/httpd/modules/libphp.so
     ```
   - Add PHP handling:
     ```apache
     <FilesMatch \.php$>
         SetHandler application/x-httpd-php
     </FilesMatch>
     ```
3. Restart Apache:
   ```bash
   brew services restart httpd
   ```

### On Windows
- PHP is included with XAMPP. Ensure the PHP module is enabled in the XAMPP Control Panel.

## Step 4: Clone the WordPress Repository
1. Install Git if not already installed:
   - Ubuntu: `sudo apt install git`
   - macOS: `brew install git`
   - Windows: Download from https://git-scm.com/downloads or use XAMPP’s Git module.
2. Navigate to the web server’s document root:
   - Ubuntu: `cd /var/www/html`
   - macOS: `cd /usr/local/var/www`
   - Windows (XAMPP): `cd C:\xampp\htdocs`
3. Clone the WordPress repository:
   ```bash
   git clone https://github.com/yourusername/my-wp-site.git
   ```
   - Replace `https://github.com/yourusername/my-wp-site.git` with your repository URL.
4. Set permissions (Ubuntu/macOS):
   ```bash
   sudo chown -R www-data:www-data /var/www/html/my-wp-site
   sudo chmod -R 755 /var/www/html/my-wp-site
   ```
   - For macOS, replace `www-data` with `_www`.

## Step 5: Configure Apache to Serve the WordPress Site
1. **Ubuntu**:
   - Create a virtual host configuration:
     ```bash
     sudo nano /etc/apache2/sites-available/my-wp-site.conf
     ```
   - Add the following, adjusting paths as needed:
     ```apache
     <VirtualHost *:80>
         ServerName my-wp-site.local
         DocumentRoot /var/www/html/my-wp-site
         <Directory /var/www/html/my-wp-site>
             AllowOverride All
             Require all granted
         </Directory>
         ErrorLog ${APACHE_LOG_DIR}/my-wp-site-error.log
         CustomLog ${APACHE_LOG_DIR}/my-wp-site-access.log combined
     </VirtualHost>
     ```
   - Enable the site:
     ```bash
     sudo a2ensite my-wp-site.conf
     sudo a2enmod rewrite
     sudo systemctl restart apache2
     ```
   - Add a local domain to `/etc/hosts`:
     ```bash
     sudo nano /etc/hosts
     ```
     - Add: `127.0.0.1 my-wp-site.local`

2. **macOS**:
   - Edit Apache config (`/usr/local/etc/httpd/httpd.conf`) to include the virtual host:
     ```apache
     <VirtualHost *:8080>
         ServerName my-wp-site.local
         DocumentRoot /usr/local/var/www/my-wp-site
         <Directory /usr/local/var/www/my-wp-site>
             AllowOverride All
             Require all granted
         </Directory>
     </VirtualHost>
     ```
   - Restart Apache:
     ```bash
     brew services restart httpd
     ```
   - Update `/etc/hosts`:
     ```bash
     sudo nano /etc/hosts
     ```
     - Add: `127.0.0.1 my-wp-site.local`

3. **Windows (XAMPP)**:
   - Edit `C:\xampp\apache\conf\extra\httpd-vhosts.conf`:
     ```apache
     <VirtualHost *:80>
         ServerName my-wp-site.local
         DocumentRoot "C:/xampp/htdocs/my-wp-site"
         <Directory "C:/xampp/htdocs/my-wp-site">
             Options Indexes FollowSymLinks
             AllowOverride All
             Require all granted
         </Directory>
     </VirtualHost>
     ```
   - Update `C:\Windows\System32\drivers\etc\hosts` (run Notepad as Administrator):
     ```text
     127.0.0.1 my-wp-site.local
     ```
   - Restart Apache via the XAMPP Control Panel.

## Step 6: Configure WordPress
1. Copy `wp-config-sample.php` to `wp-config.php`:
   ```bash
   cp /var/www/html/my-wp-site/wp-config-sample.php /var/www/html/my-wp-site/wp-config.php
   ```
2. Edit `wp-config.php` with your database details:
   ```php
   define('DB_NAME', 'my_wp_site');
   define('DB_USER', 'root');
   define('DB_PASSWORD', 'yourpassword');
   define('DB_HOST', 'localhost');
   ```
   - Replace `my_wp_site` with your database name and `yourpassword` with your MySQL root password.

## Step 7: Create and Import the Database
1. Create a MySQL database:
   ```bash
   mysql -u root -p
   ```
   ```sql
   CREATE DATABASE my_wp_site;
   EXIT;
   ```
2. Import the database backup:
   - Ensure you have the `my-wp-site.sql` file in a known location (e.g., `~/backups/my-wp-site.sql`).
   - Import using:
     ```bash
     mysql -u root -p my_wp_site < ~/backups/my-wp-site.sql
     ```
   - Or use phpMyAdmin:
     - Access `http://localhost/phpmyadmin`.
     - Select the `my_wp_site` database, go to the “Import” tab, and upload `my-wp-site.sql`.

3. Update Site URLs (if needed):
   - If the database contains URLs from another environment (e.g., a live site), update them:
     ```sql
     mysql -u root -p
     USE my_wp_site;
     UPDATE wp_options SET option_value = 'http://my-wp-site.local' WHERE option_name IN ('siteurl', 'home');
     EXIT;
     ```
   - Alternatively, use a plugin like **WP Migrate DB** to handle URL replacements during import.

## Step 8: Run the WordPress Site
1. Open a browser and visit `http://my-wp-site.local` (or `http://my-wp-site.local:8080` on macOS).
2. Log in to WordPress using the credentials from the imported database (e.g., via `wp-admin`).
3. Verify the site loads correctly and content matches the imported database.

## Troubleshooting
- **Apache Errors**: Check logs (`/var/log/apache2/error.log` on Ubuntu, `/usr/local/var/log/httpd/error_log` on macOS, or XAMPP logs).
- **Database Connection Issues**: Verify `wp-config.php` credentials and ensure MySQL is running (`sudo systemctl status mysql`).
- **Permissions**: Ensure the web server user (`www-data` on Ubuntu, `_www` on macOS, or SYSTEM on Windows) has read/write access to the WordPress directory.
- **Missing `.htaccess`**: If WordPress permalinks don’t work, create an `.htaccess` file in the WordPress root:
  ```htaccess
  # BEGIN WordPress
  <IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.php$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.php [L]
  </IfModule>
  # END WordPress
  ```

## Best Practices
- **Backup Regularly**: Back up your database before major changes (`mysqldump -u root -p my_wp_site > backup.sql`).
- **Use WP-CLI**: Install WP-CLI (https://wp-cli.org/) for faster WordPress management:
  ```bash
  wp db export backup.sql
  wp db import backup.sql
  ```
- **Secure Your Site**: Avoid using `root` for MySQL in production; create a dedicated user:
  ```sql
  CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'securepassword';
  GRANT ALL PRIVILEGES ON my_wp_site.* TO 'wp_user'@'localhost';
  FLUSH PRIVILEGES;
  ```
- **Sync Database**: Use WP Migrate DB Pro for easier database syncing across environments.

For more details:
- Apache: https://httpd.apache.org/docs/
- MySQL: https://dev.mysql.com/doc/
- WordPress Git Workflow: https://www.cloudways.com/blog/wordpress-github/