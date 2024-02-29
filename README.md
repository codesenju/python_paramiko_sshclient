# Intro to Python SSH Client Paramiko

## Prerequisites
- [Docker](https://docs.docker.com/engine/install/)
- [docker-compose](https://docs.docker.com/compose/install/)
- [python3](https://www.python.org/downloads/), [paramiko](https://www.paramiko.org/installing.html) python module
  
## What is Pramiko
Paramiko is a one of many python modules that implements the SSHv2 protocol. With paramiko you can perform various remote tasks like running commands on remote server, copying files to and from a remote server to your local machine just to mention a few. To get a full list of all the functions read more on the [SFTP client object](https://docs.paramiko.org/en/stable/api/sftp.html).

# Let's have look at our code:
## docker-compose.yml
```yaml
---
version: "3"
services:
  openssh-server:
    image: lscr.io/linuxserver/openssh-server
    container_name: lab
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Africa/Johannesburg
      - USER_PASSWORD=passw0rd
      - USER_NAME=codesenju
      - PASSWORD_ACCESS=true 
      - HOSTNAME=lab
    ports:
      - 2222:2222
    restart: unless-stopped
```

### main.py:
First we import the paramiko module on the first line than define our constants: hostname, username, paswword, PORT and ssh Client Object.

```python
import paramiko

hostname = 'localhost'
username = 'codesenju'
password = 'passw0rd'
PORT = '2222'
ssh = paramiko.SSHClient()
```

### execRemoteCommand Function:
We define a function that will take in the command and ssh client object as parameters. It will output the stdout or stderr to our console.
```python
def execRemoteCommand(cmd_, client):
    print(cmd_)
    stdin, stdout, stderr = client.exec_command(cmd_)
    output = stderr.readlines()
    output_line = ''.join(output)
    print(output_line)
    output = stdout.readlines()
    output_line = ''.join(output)
    print(output_line)
```
### Open connection:
Here we will try to open our connection with the lab container.
```python
if __name__ == "__main__":

    try:
        ssh_client = paramiko.SSHClient()
        ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh_client.connect(hostname=hostname, username=username, password=password, port=PORT)
        sftp_client = ssh_client.open_sftp()
```

### Change path on remote server:
```python
        # 1. CHANGE PATH ON THE REMOTE SERVER
        print("##########-1.-CHANGE-PATH-##########")
        sftp_client.chdir("/tmp")
        print(sftp_client.getcwd())
```
### Run commands remotely:
```python
        # 2. RUN COMMAND ON REMOTE SERVER
        print("##########-2.-REMOTE-COMMAND-##########")
        execRemoteCommand("ls -l /tmp", ssh_client)
```

###  Copy file to remote Server:
```python
        # 3. COPY FILE TO REMOTE SERVER
        print("##########-3.-REMOTE-COMMAND-##########")
        sftp_client.put("README.md", "README.md")
        execRemoteCommand("stat /tmp/README.md", ssh_client)
```
### Download file from remote server to local machine:
```python
        # 4. DOWNLOAD FILE FROM REMOTE SERVER
        print("##########-4.-DOWNLOAD-FILE-##########")
        sftp_client.get("/tmp/remote_file.txt","remote_file.txt")
```
### Handling errors and close sftp, ssh client objects:
```python
    except Exception as err:
        print("SSH CLIENT ERROR: {}".format(err))
    finally:
        sftp_client.close()
        ssh_client.close()
```

### Our *main.py* will look like this in the end:
```python
import paramiko

hostname = 'localhost'
username = 'codesenju'
password = 'passw0rd'
PORT = '2222'
ssh = paramiko.SSHClient()


def execRemoteCommand(cmd_, client):
    print(cmd_)
    stdin, stdout, stderr = client.exec_command(cmd_)
    output = stderr.readlines()
    for index, item in enumerate(output):
        print(item)
    output = stdout.readlines()
    for index, item in enumerate(output):
        print(item)


if __name__ == "__main__":

    try:
        ssh_client = paramiko.SSHClient()
        ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh_client.connect(hostname=hostname, username=username, password=password, port=PORT)
        sftp_client = ssh_client.open_sftp()

        # 1. CHANGE PATH ON REMOTE SERVER
        print("##########-1.-CHANGE-PATH-##########")
        sftp_client.chdir("/tmp")
        print(sftp_client.getcwd())

        # 2. RUN COMMAND ON REMOTE SERVER
        print("##########-2.-REMOTE-COMMAND-##########")
        execRemoteCommand("ls -l /tmp", ssh_client)

        # 3. COPY FILE TO REMOTE SERVER
        print("##########-3.-REMOTE-COMMAND-##########")
        sftp_client.put("README.md", "README.md")
        execRemoteCommand("stat /tmp/README.md", ssh_client)

        # 4. DOWNLOAD FILE FROM REMOTE SERVER
        print("##########-4.-DOWNLOAD-FILE-##########")
        sftp_client.get("/tmp/remote_file.txt","remote_file.txt")
        
    except Exception as err:
        print("SSH CLIENT ERROR: {}".format(err))
    finally:
        sftp_client.close()
        ssh_client.close()

```

# Tutorial
In this example we will be using paramiko to perform 4 remote tasks:
  1.  Change paths on remote server and return the current path.
  2.  Run any shell commands on a remote sever.
  3.  Copy a file to remote server.
  4.  Download a file form remote server to our local machine.

## a) Let's  start our remote server container called lab.
```shell
docker-compsoe up -d
```
Let's create a file on the remote server that we will be using later:
```shell
docker exec -ti lab bash -c 'echo "Hello from $HOSTNAME | $TZ" > /tmp/remote_file.txt'
```
check if the file is there:
```shell
docker exec -ti lab bash -c 'stat  /tmp/remote_file.txt'
```
## b) Run the script
Before running the script we need to make sure we have the python paramiko module installed:
```shell
pip3 install paramiko
```
Execute script:
```shell
python3 main.py
```
## Video:
![paramiko_video](media/paramiko_vid.gif)
# References
- https://codefather.tech/blog/if-name-main-python/
- https://hub.docker.com/r/linuxserver/openssh-server
- https://docs.paramiko.org/en/stable/api/client.html
