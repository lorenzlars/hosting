# Hosting

Create `.envrc` file
```
export NAS_IP=
export NAS_USER=
export NAS_PASSWORD=
export CLOUDFLARE_DNS_API_TOKEN=
```

## Initial run
Use `-K` to get a prompt for the sudo password. After the first run this is no longer needed because ansible will remove the need of the password in the `02_playbook_login.ansible.yml` playbook.
```
ansible-playbook -i hosts.ini playbooks/00_apply.yml -K
```

## After first run
```
ansible-playbook -i hosts.ini playbooks/00_apply.yml
```
