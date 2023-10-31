# Setting up monitoring server
```
sudo ansible-playbook -i monitor_inventory.ini playbooks/monitor_install_prometheus_grafana.yml -u svc_admin --private-key=/root/.ssh/id_rsa
sudo ansible-playbook -i monitor_inventory.ini playbooks/monitor_install_pf.yml -u svc_admin --private-key=/root/.ssh/id_rsa
```

# Set up the target servers
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
ansible-playbook -i inventory.ini -l atl.i.rm.vg playbooks/monitor_host.yml -u root --private-key=~/.ssh/id_rsa
```
