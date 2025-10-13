## Usage Instructions

1. **Preparing the Environment:**
   - Ensure that you have Ansible installed on your control node.
   - Create a `.ansible` directory to store sensitive information (vault password).
     `echo "your-vault-password" > ~/.ansible/vault_pass.txt`
   - Encrypting & Decrypting Sensitive Data
     `ansible-vault encrypt group_vars/homelab.yml`
     `ansible-vault decrypt group_vars/homelab.yml`
   - Associating the Diff Tool with Vault Files
     `git config diff.ansible-vault.textconv "ansible-vault view"`

2. **Inventory and Configuration:**
   - Edit `hosts` file to list all your extant hosts.
   - Adjust `group_vars` and `host_vars` to suit your environment.

3. **Running Playbooks:**
   - Use the following command to execute a playbook:
     ```bash
     ansible-playbook -i hosts playbooks/homepi.yml
     ```
   - Replace `homepi.yml` with the relevant playbook file for your needs.

## Adding pre commit hook for safe commit

1. **Navigate to the Git Hooks Directory:**

   Go to the `.git/hooks` directory in your project repository.

2. **Create the Pre-commit Hook Script:**

   Create a new file named `pre-commit` in the hooks directory with the following content:

   ```bash
   #!/bin/bash

   # Function to check if a file is encrypted by Ansible Vault
   is_encrypted() {
       local file="$1"
       head -n 1 "$file" | grep -q -- "^\$ANSIBLE_VAULT"
   }

   # Gather a list of files staged for commit
   FILES=$(git diff --cached --name-only)

   # Loop through each file and check if it is an unencryption Ansible var file
   for file in $FILES; do
       if [[ "$file" =~ ^(group_vars|host_vars)/ ]]; then
           if ! is_encrypted "$file"; then
               echo "Error: $file is not encrypted with Ansible Vault."
               echo "Please encrypt it before committing."
               exit 1
           fi
       fi
   done

   # If we reach here, all checks passed
   exit 0
   ```

3. **Make the Script Executable:**

   Ensure the script is executable by running:

   ```bash
   chmod +x .git/hooks/pre-commit
   ```

## Note

DONOT commit plain text passwords, always encrypt and commit even on local. Git pre commit hooks will notify you when you do so.
