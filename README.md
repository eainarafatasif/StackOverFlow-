## StackOver Flow - Clone

<a href="https://github.com/Yawan-1/StackOverFlow--Clone/stargazers"><img alt="GitHub stars" src="https://img.shields.io/github/stars/Yawan-1/StackOverFlow--Clone"></a>
<a href="https://github.com/Yawan-1/StackOverFlow--Clone/blob/master/LICENSE"><img alt="GitHub license" src="https://img.shields.io/github/license/Yawan-1/StackOverFlow--Clone"></a>
<img alt="GitHub commit activity" src="https://img.shields.io/github/commit-activity/m/Yawan-1/StackOverFlow--Clone">
![python3.x](https://img.shields.io/badge/python-3.x-brightgreen.svg)

Clone of Stack Overflow where I implemented nearly all of its functionalities. My intention was to provide insight and demonstration to developers on the inner workings of Stack Overflow - including how tasks are performed <b style="color:lightgreen">behind the scenes</b> and how queries are executed..

> Note: Please have a look at the Blog explaining <a href="https://yawan-1.github.io/">What I learned from this Project?</a>

## Images

<img src="/images/animation.gif">

## Demo

Here is a working live demo : <a href="https://stonkoverflow.herokuapp.com/">Demo</a> <b>(Removed from heroku because usage of so's production LOGO</b>)</b>

## Technology Stack

* [Python 3.7.x](https://www.python.org/)
* [Django Web Framework 3.2.x](https://www.djangoproject.com/)
* [Redis 5.x](https://pypi.org/project/django-redis/)
* [BootStrap 4](https://getbootstrap.com/)
* [Jquery 3](https://api.jquery.com/)
* [Postgresql 14](https://www.postgresql.org/)


## Functionalities


* 50+ Badges are implemented to award
* 20 Privileges to Earn
* Track Badges
* Reputation Awarding
* Privilege and Activity Notifications
* Live Q&A MarkDown Preview
* User @mentioning in comments
* Create and award Bounties
* <code>Threading</code> to keep track of the remaining days of Bounty.
* Reviewing Tasks :
  * First Question Review
  * First Answer Review
  * Late Answer Review
  * Review Flag Posts
  * Review Flag Comments
  * Review Close Votes
  * Review ReOpen Votes
  * Review Low-Quality Posts
  * Review Suggested Edits


* And much more. You can find list of all functionalities <a href="https://github.com/Yawan-1/StackOverFlow--Clone/blob/759157fc68f59398d9352ddd705eee396336bb81/Functionalities.md">Here</a>

To deploy your Django project from GitHub using Nginx, MySQL, and an SSL certificate on an Ubuntu server, follow these steps:

### Prerequisites
- An Ubuntu server with root or sudo access.
- A domain name pointing to your server's IP address.
- A GitHub repository containing your Django project.
- A virtual environment set up for your project.
  
### 1. **Update and Install Dependencies**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install python3-pip python3-dev libpq-dev nginx curl git
   sudo apt install mysql-server libmysqlclient-dev
   ```
Once all the packages are installed, you can start the Apache service and configure it to run on startup by entering the following commands:

```bash
 systemctl start Nginx
 systemctl enable Nginx

```
Now Allow the firewall
```bash
ufw allow 80
ufw allow 443

```

### 2. **Clone Your GitHub Repository**
   ```bash
   cd /var/www/html/
   sudo git clone https://github.com/unfoldadmin/django-unfold.git
   cd django-unfold
   ```

### 3. **Set Up Virtual Environment**
   ```bash
   sudo apt install python3-venv
   python3 -m venv projectenv
   source projectenv/bin/activate
   pip install django
   pip install mysqlclient
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

### 4. **Configure Django Settings**
- Update your `settings.py` to include your domain in the `ALLOWED_HOSTS`:
   ```python
   ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']
   ```
- Ensure your `DATABASES` setting points to your MySQL database.

### 5. **Set Up MySQL Database**
   ```bash
   sudo apt install mysql-server
   sudo mysql_secure_installation
   ```
   - Log in to MySQL and create a database and user:
     ```bash
     sudo mysql -u root -p
     ```
     ```sql
     CREATE DATABASE django_unfold;
     CREATE USER 'youruser'@'localhost' IDENTIFIED BY 'yourpassword';
     GRANT ALL PRIVILEGES ON django_unfold.* TO 'youruser'@'localhost';
     FLUSH PRIVILEGES;
     EXIT;
     ```
   - Update your Django `settings.py` to connect to the MySQL database.

  ```bash
  DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'your_db_name',
        'USER': 'your_db_user',
        'PASSWORD': 'your_db_password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}

 ```

### 6. **Run Migrations and Collect Static Files**
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   python manage.py collectstatic
   ```

### 7. **Set Up Gunicorn**
   ```bash
   pip install gunicorn
   gunicorn --bind 0.0.0.0:8000 djangoproject.wsgi
   ```
Next, you need to create a systemd service file to manage the Django. First, create a Gunicorn socket file.
 ```bash
    nano /etc/systemd/system/gunicorn.socket
```
Add the following configuration.
  ```bash 
        [Unit]
        Description=gunicorn socket
        [Socket]
        ListenStream=/run/gunicorn.sock
        [Install]
        WantedBy=sockets.target
   ```
   - Create a system service file for Gunicorn:
     ```bash
     sudo nano /etc/systemd/system/gunicorn.service
     ```
     - Add the following:
       ```bash
       [Unit]
        Description=gunicorn daemon
        Requires=gunicorn.socket
        After=network.target
        [Service]
        User=root
        Group=www-data
        WorkingDirectory=/root/project
                ExecStart=/root/project/djangoenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock djangoproject.wsgi:application
            [Install]
        WantedBy=multi-user.target
       ```
     - Enable and start the Gunicorn service:
       ```bash
       sudo systemctl start gunicorn
       sudo systemctl enable gunicorn
       systemctl start gunicorn.socket
       systemctl enable gunicorn.socket
       
       ```

### 8. **Configure Nginx**
   - Create an Nginx server block:
     ```bash
     sudo nano /etc/nginx/sites-available/django_unfold
     ```
     - Add the following:
       ```nginx
       server {
           listen 80;
           server_name yourdomain.com www.yourdomain.com;

           location = /favicon.ico { access_log off; log_not_found off; }
           location /static/ {
               root /var/www/html/django-unfold;
           }

           location / {
               include proxy_params;
               proxy_pass http://unix:/var/www/html/django-unfold/gunicorn.sock;
           }
       }
       ```
   - Enable the configuration and restart Nginx:
     ```bash
     sudo ln -s /etc/nginx/sites-available/django_unfold /etc/nginx/sites-enabled
     sudo nginx -t
     sudo systemctl restart nginx
     ```

Ensure Correct Permissions for Static and Media Files
If you have static and media directories outside of the main project directory, set their ownership and permissions similarly:

```bash
sudo chown -R www-data:www-data /var/www/html/django-unfold/
sudo chmod -R 755 /var/www/html/django-unfold/

```

### 9. **Obtain an SSL Certificate**
   - Install Certbot:
     ```bash
     sudo apt install certbot python3-certbot-nginx
     ```
   - Obtain and install the SSL certificate:
     ```bash
     sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
     ```
   - Follow the prompts to complete the SSL installation. Certbot will automatically configure your Nginx server block to use SSL.

### 10. **Testing**
   - Visit `https://yourdomain.com` to ensure everything is working correctly.

### 11. **Set Up Automatic Renewals for SSL**
   ```bash
   sudo systemctl status certbot.timer
   ```

This setup will get your Django project deployed with Nginx, MySQL, and an SSL certificate on Ubuntu.


# Contributors
Contributions are welcome, and they are greatly appreciated! Every little bit helps, and credit will always be given.<br/><br/>

Please star the repo and feel free to make pull requests. <br/><br/>
<a href='https://ko-fi.com/J3J617AIN' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://az743702.vo.msecnd.net/cdn/kofi4.png?v=2' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>



## Contributing

If you have any question or issues, It may have bugs that i may have missed. You can create <a href="https://github.com/Yawan-1/StackOverFlow--Clone/pulls">Pull request</a>.

Note: <small>Frontend and complete design is also inside this project's repo (html, css).</small>

