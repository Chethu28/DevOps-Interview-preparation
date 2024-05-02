1. **ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt** ------> Ansible will automatically search for the password in that file
2. **ansible-vault create passwd.yml** ------> Create a new encrypted data file.Set the password for vault
3. **ansible-vault edit passwd.yml** ------> Edit encrypted file
4. **ansible-vault rekey passwd.yml** -----> Change password for encrypted file

5. **ansible-playbook playbooks/atmo_playbook.yml -e "ATMOUSERNAME=atmouser"** -----> Passing Variables via CLI

6. **ansible-playbook playbooks/PLAYBOOK_NAME.yml --check** ----> Running a playbook in dry-run mode

7. **ansible-playbook playbooks/atmo_playbook.yml --user atmouser** -----> Specifying a user

8. **ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"**
9. **ansible-playbook arcade.yml --extra-vars '{"pacman":"mrs","ghosts":["inky","pinky","clyde","sue"]}'**
10. **ansible-playbook release.yml --extra-vars "@some_file.json"**
