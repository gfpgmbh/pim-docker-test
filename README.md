## Test Docker environment of the PIM application
<p>The instruction describes the entire process of installing a PIM application on a clean Debian 11 system that has just been installed.</p>

### Preparation

<p>First of all, you need to check if current user is sudoer.</p>

```
su
```

<p>Enter the root-password and then:</p>

```
nano /etc/sudoers
```

<p>Block "User privilege specification" should look like this:</p>

```
...
# User privilege specification
root    ALL=(ALL:ALL) ALL
lpim    ALL=(ALL:ALL) ALL
...
```

<p>To Save and Exit press: Ctrl+o, Enter, Ctrl+x.</p>

```
sudo apt-get update
```

> If there are any errors, then you need to disable the local repository by commenting out the first lines pointing to the CD-ROM. Otherwise, ignore this step.
>
> sudo nano /etc/apt/sources.list

### Docker installation

<p>Install required applications, keys and repository.</p>

```
sudo apt-get install ca-certificates curl gnupg lsb-release
```
```
sudo curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
```
sudo echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

<p>Then, you need to update all packages.</p>

```
sudo apt-get update
```

<p>Finally install Docker.</p>

```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
### Docker-compose installation

<p>Additionally, you need to install docker-compose.</p>

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

<p>Set access rights for application execution.</p>

```
sudo chmod +x /usr/local/bin/docker-compose
```

<p>Adding a local user "lpim" to the docker group so that there are no problems accessing the socket. If the user is not "lpim", then replace with the required value.</p>

```
sudo usermod -aG docker lpim
```

<p>For the changes to take effect, you may have to close the console and log in again.</p>

### Docker authentication

```
docker login -u gfpgmbh
```

<p>Next, instead of a password, you must enter a token for clients that has read-only access rights.</p>

### Copying configuration files from a remote repository

```
git clone https://github.com/gfpgmbh/pim-docker-test.git
```

<p>If <strong>git</strong> is not installed, then you need to install it additionally (apt-get install git).</p>

```
cd pim-docker-test/db
```

<p>Creating a file with environment variables.</p>

```
nano .env
```

### Content of .env file, you need to insert this
```
MYSQL_DATABASE=pim
MYSQL_ROOT_PASSWORD=root-password-replace-me
MYSQL_PORT=33066

MYSQL_USER=pim
MYSQL_PASSWORD=user-password-replace-me

PMA_PORT=8080
```

<p>It is necessary to set the correct <strong>MYSQL_ROOT_PASSWORD</strong> and <strong>MYSQL_PASSWORD</strong> passwords.</p>

<p>Next, to Save and Exit press: Ctrl+o, Enter, Ctrl+x.</p>

### Starting the database service

<p>Creating an external bridge-type network.</p>

```
docker network create mysql-network
```

<p>Creating and running a container in the background.</p>

```
docker-compose up -d
```

### Setting a password for the application config/pim.cfg

```
cd ../app/config
```
```
nano pim.cfg
```

<p>Replace the standard password with the one that was previously specified as <strong>MYSQL_PASSWORD</strong> in .env file.</p>

<p>Next, to Save and Exit press: Ctrl+o, Enter, Ctrl+x.</p>

<p>Set access rights to the sessions folder.</p>

```
chmod -R 777 ../repo
```

### Starting the Application service

```
cd ..
```

<p>Creating and running a container in the background.</p>

```
docker-compose up -d
```

### Done. Links to websites: (replace 127.0.0.1 by required)

http://127.0.0.1/pim/ - PIM Application

http://127.0.0.1:8080 - PhpMyAdmin, DB Administration Tool

### Aliases

<p>In order for aliases or short commands to be available, you need to add them to the <strong>/home/lpim/.bashrc</strong> file.</p>

```
nano /home/lpim/.bashrc
```

<p>Add the following text to the very end of the file.</p>

```
alias pim-down="docker-compose -f /home/lpim/pim-docker-test/app/docker-compose.yml down"
alias pim-stop="docker-compose -f /home/lpim/pim-docker-test/app/docker-compose.yml stop"
alias pim-clear-repo="sudo rm -rf /home/lpim/pim-docker-test/app/repo/*"
alias pim-up="docker-compose -f /home/lpim/pim-docker-test/app/docker-compose.yml up -d"
alias pim-update="docker-compose -f /home/lpim/pim-docker-test/app/docker-compose.yml down ; docker-compose -f /home/lpim/pim-docker-test/app/docker-compose.yml rm -f ; docker-compose -f /home/lpim/pim-docker-test/app/docker-compose.yml pull ; docker-compose -f /home/lpim/pim-docker-test/app/docker-compose.yml up -d"
```

<p>To Save and Exit press: Ctrl+o, Enter, Ctrl+x.</p>

<p>For the changes to take effect, you may have to close the console and log in again.</p>
