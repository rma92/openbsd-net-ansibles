# openbsd-net-ansibles
Set Up Openbsd Ansibles

## Old versions of OpenBSD - set install URL to something with the packages
```
echo "https://ftp.lysator.liu.se/pub/OpenBSD" > /etc/installurl
```
(On vultr's install, the default content is "https://cdn.openbsd.org/pub/OpenBSD",  however old versions are not kept on OpenBSD's FTP)

## Add a root ssh key for using ansible
### Create a key (as your normal user)
```
cd ~/.ssh
ssh-keygen
```
### Copy the key - SCP a script
Create a script (addkey.sh)
```
mkdir -p /root/.ssh
echo '<CONTENTS OF ID_RSA.PUB>' >> /root/.ssh/authorized_keys
chown -R root /root/.ssh
```
Replace the `<CONTENTS OF ID_RSA.PUB>` with the actual contents of the generated ID_RSA.PUB.

ssh the script to the host you want to add the key.  Run it as root (`sudo sh addkey.sh` or otherwise).
### Copy the key - paste a oneliner
Replace the contents of `<CONTENTS OF ID_RSA.PUB>`, but paste this into all the machine's terminals, enter sudo password, you're good to go.
```
sudo sh -c 'mkdir -p /root/.ssh && echo "<CONTENTS_OF_ID_RSA.PUB>" >> /root/.ssh/authorized_keys && chown -R root /root/.ssh'
```

### Log in to ssh with the key.
```
ssh -i ~/.ssh/id_rsa root@<hostname>
```

## Set up Ansible
Configuring OpenBSD servers using Ansible is a great way to automate and manage your infrastructure. The steps you've outlined are a good start, but there are a few more details to consider. Here's a more comprehensive guide on how to get started:

1. **Install Ansible**:

   You should install Ansible on a control machine, not on the servers you want to manage. Your control machine can be a separate server or your local development machine. You can install Ansible on OpenBSD using `pkg_add ansible` as you mentioned.

   ```sh
   pkg_add ansible
   ```

2. **Create SSH Key Pair**:

   It's a good practice to create an SSH key pair for the user on your control machine (not necessarily root). You can use a non-root user with sudo privileges for security reasons.

   ```sh
   ssh-keygen
   ```

   Follow the prompts to generate the key pair. This will create `~/.ssh/id_rsa` (private key) and `~/.ssh/id_rsa.pub` (public key) by default.

3. **Distribute Public Key**:

   Copy the public key (`~/.ssh/id_rsa.pub`) from your control machine to each of the OpenBSD servers you want to manage. You can use the `ssh-copy-id` command to automate this process:

   ```sh
   ssh-copy-id username@server_ip
   ```

   Replace `username` with your username on the remote server and `server_ip` with the IP address or hostname of the server. This command will copy your public key to the remote server's `~/.ssh/authorized_keys` file.

4. **Ansible Inventory**:

   Create an Ansible inventory file (usually named `inventory`) to list the servers you want to manage. Here's an example:

   ```ini
   [openbsd_servers]
   server1 ansible_ssh_host=server1_ip
   server2 ansible_ssh_host=server2_ip
   # Add more servers as needed
   ```

   Replace `server1`, `server2`, and `serverX` with your server names or aliases, and `server1_ip`, `server2_ip`, and `serverX_ip` with their corresponding IP addresses.

5. **Ansible Configuration**:

   You can customize Ansible's behavior by creating an `ansible.cfg` file, but it's optional. The default settings should work for most cases.

6. **Create Ansible Playbooks**:

   Now you can create Ansible playbooks to define the configuration tasks you want to automate on your OpenBSD servers. Ansible playbooks are written in YAML format and describe the desired state of your servers.

7. **Run Ansible Playbooks**:

   To apply your configuration to the managed servers, run Ansible playbooks using the `ansible-playbook` command:

   ```sh
   ansible-playbook -i inventory my_playbook.yml
   ```

   Replace `my_playbook.yml` with the filename of your playbook.

8. **Testing and Iteration**:

   Test your playbooks on a single server or a small group of servers before applying them to all your servers to ensure they work as expected. Iterate and refine your playbooks as needed.

By following these steps, you'll be able to use Ansible to manage your OpenBSD servers effectively. Using a non-root user with sudo privileges is recommended for security, but if you decide to use the root user, exercise caution and ensure that your Ansible control machine is secure.
