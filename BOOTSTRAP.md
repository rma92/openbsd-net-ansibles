# OpenBSD Bootstrap With Ansible
To set up an appropriate security posture yet allow for configuration, we will:
* Create an svc_admin user.
* svc_admin can only log in with public key.
* svc_admin can sudo as root passwordless.
* svc_admin has a password.  Generate the password hash with the `encrypt` command on OpenBSD, paste it into the fault in the variable name svc_admin_password

## Preliminary steps to use
* Set up the vault's contents (see next section)
* Put svc_admin public key in playbooks/files/svc_admin_rsa_key.pub.
```
sudo cp /root/.ssh/id_rsa.pub playbooks/files/svc_admin_rsa_key.pub
sudo chown `whoami` playbooks/files/svc_admin_rsa_key.pub
```

## Set up the Vault's Contents:
Use the encrypt command to get the password hash:
```shell
encrypt
<Enter the Password and hit enter>
```
Copy the hash.

```shell
ansible-vault create playbooks/svc_deploy_vault.yml
```
Enter a vault password, and set svc_admin_password_hash in the vault file.
```
svc_admin_password_hash: "Your_Password_Here"
```

To edit the vault later, you can run
```
ansible-vault edit playbooks/svc_deploy_vault.yml
```

## Run the playbooks
```
sudo ansible-playbook -u root --private-key=/root/.ssh/id_rsa -i monitor_inventory.ini playbooks/bootstrap_all_install_python.yml
sudo ansible-playbook -u root --private-key=/root/.ssh/id_rsa -i monitor_inventory.ini playbooks/bootstrap_svc_admin.yml --ask-vault-password
```
If you want to manually specify the private key and user instead:
```
ansible-playbook -i monitor_inventory.ini playbooks/bootstrap_svc_admin.yml -u root --private-key=/home/root/id_rsa
```

## Results
After running the second playbook, you can ssh to the host using root's SSH key, then run sudo without a password to be root.
```
sudo ssh -i /root/.ssh/id_rsa svc_admin@<hostname>
sudo whoami
```
Consequently, other ansible playbooks can be run as that user:
```
sudo ansible-playbook -i monitor_inventory.ini playbooks/test_whoami_check.yml -u svc_admin --private-key=/root/.ssh/id_rsa
sudo ansible-playbook -i monitor_inventory.ini playbooks/test_whoami_become_check.yml -u svc_admin --private-key=/root/.ssh/id_rsa
```
