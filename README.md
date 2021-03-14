## Create Instance
1. AWS > EC2 > Launch Instance.
2. Select **Amazon Linux 2 AMI (HVM), SSD Volume Type (64-bit (x86))**.
3. Select free version in terms of size.
4. Edit security groups and `Add Rule`. Set it to Type="`HTML`", Procotol="`TCP`" Port=`80`, Source="`Anywhere`" , then the CIDR field (the next one over) to `0.0.0.0/0, ::/0`. This is needed for Django apps because NGINX needs this port to operate. THEN, select `Add Rule` again and set it to Type="`CUSTOM TCP`", Procotol="`TCP`" Port=`8000`, Source="`Anywhere`", then the CIDR field (the next one over) to `0.0.0.0/0, ::/0`. This is needed for the view for the Django app.
5. For creating a new key pair, create one ex. "jdango-key" and then **Download Key**.
6. Once downloaded and moved to under your root `/~` directory, then **Launch Instance**.

## Connect to Instance
1. Select your instance from AWS's dashboard of running instances then select **Connect**.
2. Follow the directions to connect to the AWS instance using **SSH Client**.
3. Now SSH to it: `ssh -i "django-key.pem" ec2-user@ec2-34-201-63-11.compute-1.amazonaws.com`.
4. When connecting to it the first time, to establish fingerprint connecting your key to AWS, type "yes" when prompted.
5. That's it! If you verify you see the AWS text image, you're in.

## Deploy Django App to EC2 Instance
1. SSH to EC2 instance: `ssh -i "django-key.pem" ec2-user@ec2-34-201-63-11.compute-1.amazonaws.com`.
2. Make sure your way of adding and updating packages is most up to date: `sudo yum update`.
3. Because we are using a minimal version of Linux, it doesn't have gcc (GNU Compiler Collection) so in order to unzip tar files and such, install it: `sudo yum install gcc -y`.
4. Install python3: `sudo yum install python3 -y`.
5. Check Python3 version: `python3 --version`. Ensure `3.7` or above.
6. See if pip installed: `python3 -m pip --version`. If not, install it using python 3 (I think `python3 get-pip.py`).
7. Create virtual environment for the project: `python3 -m venv env`.
8. Activate virtual environment: `source env/bin/activate`.
9. ONCE virtual environment is created and active, install Django: `python3 -m pip install Django`.
10. Check Django version: `python3 -m django --version`. Ensure `3.0` or above.
11. Install git: `sudo yum install git -y`.
12. While on root directory, `git init`.
13. Clone the git repo attached to the project to populate the home directory of the EC2 instance with said code repo.
ex: `git clone https://github.com/django-ve/django-helloworld`
14. Install nginx. AWS has their own you have to install: `sudo amazon-linux-extras install nginx1 -y`
15. Install gunicorn (a Python Web Server Gateway Interface HTTP server): `pip3 install gunicorn`. This acts as a WSGI interface between our Django application and the nginx server using a machanism called Unix Socket.
16. Check your SQLite version: `sqlite3 --version`. If below version 3.8.3, need to get more up to date version and do ALL of the steps found in section below: **"Upgrade SQLite version above 3.8.3"**. If it's above version `3.8.3` already or a `sudo yum upgrade sqlite3 -y` returns a version above that, proceed to following step. Reason we need to do this is because after Django version `2.1` they started requiring this more advanced version (I assume for security, but I don't know).
17. Go into project: `cd django-helloworld`.
18. Preform Django migration to propagate your changes: `python3 manage.py migrate`.
19. Go to your site's **Public IPv4 address** as it is shown on the details of your AWS EC2 instance with port 80 appended to it. Ex: `34.201.63.11:80`. Verify a page loads
20. Now check that port 8000 is working and that the project is loading by binding gunicorn to your port 8000: `gunicorn --bind 0.0.0.0:8000 helloworld.wsgi:application` then navigate to your site at that address. Ex: `http://34.201.63.11:8000/`.
  IF you get a console error, make sure you are still in the `/django-helloworld` directory.
  IF you get a DISALLOWEDHOSTS at that address, you have to `cd` into the `settings.py` for that folder `helloworld` and then add ex. `34.201.63.11` to the list of supporting hosts. Will look like: `ALLOWED_HOSTS = ['34.201.63.11', 'ec2-34-201-63-11.compute-1.amazonaws.com']`. NOTE - by adding the ec2 form of the link will also work if you prefer that over the ip address form.

<img width="670" alt="screenshot of hello world working" src="https://user-images.githubusercontent.com/7783699/111058947-f6a99300-845f-11eb-8392-6403a6787b9b.png">

### Upgrade SQLite version above 3.8.3
1. Download a recent version of the tarball and place on your system manually or via command line: `wget https://www.sqlite.org/2021/sqlite-autoconf-3350000.tar.gz`
2. Uncompress the tar file: `tar -xzf sqlite-autoconf-3350000.tar.gz
cd sqlite-autoconf-3350000/``
3. Configure it: `./configure`. If this fails due to a "no acceptable C compiler found in $PATH", it means you didn't install **GCC** from STEP 3.
4. Now that it's configured, make it: `sudo make install`.
5. Next, you need to edit the activate script used to start the virtualenv so that python looks in the right place for the newly installed SQLite. Why do you have to do this? If you do a `python3 -c "import sqlite3; print(sqlite3.sqlite_version)"` you will see the version still says ex. `3.7.17`. This fixes that. Simply add the following line at the top of the file:
`export LD_LIBRARY_PATH="/usr/local/lib"`
Do that by doing a `vi ~/env/bin/activate`, type `i` to edit the file, paste the export line, press `ESC` key, type `:wq` to leave while saving.
6. After it's added, close and reopen your SSH connection then activate your environment again: `source env/bin/activate`. Verify that doing a `python3 -c "import sqlite3; print(sqlite3.sqlite_version)"` returns a version of sqlite above `3.8.3` now.

## Git Setup (perhaps only have to do on private repo)
1. Set your local Github email.
```
git config --local user.email "{YOUR_EMAIL}"
# ex: git config --local user.email "bob.smith@gmail.com"
```
2. Set your global Github email.
```
git config --global user.email "{YOUR_EMAIL}"
# ex: git config --global user.email "bob.smith@gmail.com"
```
3. Find out what your username is on github.
4. Set your github user name.
```
git config --global user.name "{YOUR_GITHUB_NAME}"
# ex: git config --global user.name "bobsmith"
```
5. Do following and ensure user.email shows up:
```
git config --local --list
```
6. Do following and ensure user.email and user.name shows up:
```
git config --global --list
```

### Setup SSH Keys
1. Go to the location of your SSH keys.
```
cd ~/.ssh
ls
```
2. Verify you see a "id_rsa" AND "id_rsa.pub". If not, do the following:
```
ssh-keygen -t rsa -C "{YOUR_EMAIL}"
# ex: ssh-keygen -t rsa -C "bob.smith@gmail.com"
```
3. When field appears to enter name, you MUST type: `id_rsa` REMEMBER to set a password you can remember as every time you push you will need to do it!!!
4. Next, verify both now appear in there with:
```
ls
```
5. Open the file in view only and copy all that is inside of it. Should start with `"ssh"` and end with **your email**. View it by doing command:
```
less id_rsa.pub
```
6. With that copied in your clipboard, in Github, go to https://github.com/settings/keys. Select ``"New SSH Key"`` > call it whatever, ex `"rsa"` then paste that secret key to the field and save it by selecting `Add SSH Key`.
7. SSH or navigate to project location: `ssh -i "django-key.pem" ec2-user@ec2-34-201-63-11.compute-1.amazonaws.com`.
8. Populate a folder under root of our actual project: git clone git@github.com:TEAM/PROJECT.git
9. cd /helloworld
10. git pull
This is the repo! You did it!

### Potential Git Errors
If you receive email about "push declined due to email privacy restrictions"
1. To Fix, you have to change your email in your git config to the email found at: https://github.com/settings/emails about halfway down in bold ex: `1123699+bobsmith@users.noreply.github.com`
```
git config --local user.email "{THAT_EMAIL}"
# ex: git config --local user.email "1123699+bobsmith@users.noreply.github.com"
git config --global user.email "{THAT_EMAIL}"
# ex: git config --global user.email "1123699+bobsmith@users.noreply.github.com"
```
2. Now you have to recommit the changes you wanted but make those changes reflect the new email, not the old one:
```
git rebase -i
:wq
git commit --amend --reset-author
:wq
```
3. Then try pushing again: `git push`. It should work now.

## Setup RDS
TODO
but bunch of links on this:
- https://www.youtube.com/watch?v=PCjeBQ2636Y (29 minute)
- https://www.youtube.com/watch?v=3HPq12w-dww (17 minute)
- https://www.youtube.com/watch?v=udh1L1YffqQ (9 minute)

## Doing Dev Work
1. SSH to box.
2. Run: `source env/bin/activate`
3. Go to code repo directory.
4. Maybe a `sudo service nginx start` but I don't think so.
5. Then run gunicorn: `gunicorn --bind 0.0.0.0:8000 helloworld.wsgi:application` to make the changes reflected.
6. Make changes on X2GO, manually with command line vi's, or whatever.
