# VUB Hydra HPC — SSH Access Setup

This document describes how I configured SSH access to the VUB Hydra HPC cluster through the Flemish Supercomputer Center (VSC).

## 1. Prerequisites

I have:

* A VSC account
* VSC user ID: `vsc11870`
* Access to the VUB Hydra HPC cluster
* A Linux machine with OpenSSH installed

The VUB Hydra login host is:

```text
login.hpc.vub.be
```

SSH login format:

```bash
ssh vsc11870@login.hpc.vub.be
```

---

## 2. Generate an SSH key pair

I generated an RSA SSH key pair specifically for VSC:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_vsc
```

This creates two files:

```text
~/.ssh/id_rsa_vsc       # PRIVATE key — NEVER share this
~/.ssh/id_rsa_vsc.pub   # PUBLIC key — this can be uploaded to VSC
```

### Important

The private key must remain secret.

Never upload this file to GitHub:

```text
~/.ssh/id_rsa_vsc
```

The public key is safe to distribute:

```text
~/.ssh/id_rsa_vsc.pub
```

---

## 3. Add a passphrase to the SSH key

A passphrase can be added to an existing SSH private key.

```bash
ssh-keygen -p -f ~/.ssh/id_rsa_vsc
```

If the key did not previously have a passphrase, leave the old passphrase empty when prompted.

The passphrase protects the private key if someone obtains the key file.

The public key does not change when adding a passphrase.

---

## 4. Register the public key with VSC

The public key needs to be associated with the VSC account.

The public key is located at:

```text
~/.ssh/id_rsa_vsc.pub
```

The key can be added through the VSC account portal:

1. Log in to the VSC Account page.
2. Choose **Edit Account**.
3. Find **Add public key**.
4. Upload:

```text
~/.ssh/id_rsa_vsc.pub
```

5. Select **Upload extra public key**.
6. Save/update the account.
7. Verify that the key appears under the public keys.

The key may take some time to propagate to the HPC infrastructure.

---

## 5. Connect to Hydra

The SSH command is:

```bash
ssh -i ~/.ssh/id_rsa_vsc vsc11870@login.hpc.vub.be
```

Do not put `<` or `>` around the username/hostname.

Correct:

```bash
ssh -i ~/.ssh/id_rsa_vsc vsc11870@login.hpc.vub.be
```

Incorrect:

```bash
ssh -i ~/.ssh/id_rsa_vsc <vsc11870@login.hpc.vub.be>
```

---

## 6. First connection

On the first connection, SSH asks whether the server can be trusted:

```text
The authenticity of host 'login.hpc.vub.be' can't be established.

Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

After verifying the host, answer:

```text
yes
```

The server's host key is then stored in:

```text
~/.ssh/known_hosts
```

Future connections will recognize the server.

---

## 7. SSH key passphrase

When SSH uses the private key, it may ask:

```text
Enter passphrase for key '/home/<username>/.ssh/id_rsa_vsc':
```

This is the **SSH key passphrase**, not necessarily the VSC account password.

When typing the passphrase, nothing appears on the screen. This is normal.

If Enter is accidentally pressed without entering the passphrase, the authentication attempt simply fails. The SSH command can be run again.

---

## 8. Troubleshooting SSH authentication

If the connection fails with:

```text
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

use verbose SSH output:

```bash
ssh -v -i ~/.ssh/id_rsa_vsc vsc11870@login.hpc.vub.be
```

For even more detail:

```bash
ssh -vvv -i ~/.ssh/id_rsa_vsc vsc11870@login.hpc.vub.be
```

Useful lines to look for include:

```text
Offering public key
```

and:

```text
Server accepts key
```

If the server rejects the key, check that the public key registered in the VSC portal corresponds to the private key being used locally.

---

## 9. Check the SSH key fingerprint

The public key fingerprint can be displayed with:

```bash
ssh-keygen -lf ~/.ssh/id_rsa_vsc.pub
```

This is useful for verifying that the key uploaded to VSC is the same key being used locally.

The public key itself can be displayed with:

```bash
cat ~/.ssh/id_rsa_vsc.pub
```

---

## 10. SSH agent

Ubuntu may automatically use an SSH agent.

To see loaded keys:

```bash
ssh-add -l
```

If necessary, the key can be added manually:

```bash
ssh-add ~/.ssh/id_rsa_vsc
```

After entering the passphrase, the SSH agent can use the key without asking for the passphrase every time.

For troubleshooting, SSH can be forced to use only the specified key:

```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_rsa_vsc vsc11870@login.hpc.vub.be
```

---

# 11. Basic Linux navigation on Hydra

After logging in, normal Linux commands can be used.

Check the current directory:

```bash
pwd
```

List files:

```bash
ls
```

List files including hidden files:

```bash
ls -la
```

Enter a directory:

```bash
cd directory_name
```

Go one level up:

```bash
cd ..
```

Return to the home directory:

```bash
cd ~
```

---

# 12. VSC storage locations

VSC provides environment variables for important storage locations.

Check them with:

```bash
echo $VSC_HOME
echo $VSC_DATA
echo $VSC_SCRATCH
```

## VSC_HOME

```bash
cd $VSC_HOME
```

Home storage is intended for configuration and relatively small amounts of data.

## VSC_DATA

```bash
cd $VSC_DATA
```

This is intended for data that should be kept persistently.

For example:

```bash
mkdir -p $VSC_DATA/my_project
cd $VSC_DATA/my_project
```

## VSC_SCRATCH

```bash
cd $VSC_SCRATCH
```

Scratch storage is intended for temporary data used during computations.

For example:

```bash
mkdir -p $VSC_SCRATCH/my_project
cd $VSC_SCRATCH/my_project
```

Scratch should not be treated as a permanent backup location.

---

# 13. Login nodes vs compute nodes

The Hydra login node should primarily be used for:

* Managing files
* Editing code
* Compiling software
* Preparing jobs
* Submitting jobs
* Checking job status

Heavy computations should **not** normally be run directly on the login node.

Hydra uses a scheduler to allocate compute resources.

---

# 14. Interactive compute job

An interactive compute session can be started with Slurm:

```bash
srun --pty --time=00:10:00 --mem=2G --cpus-per-task=1 bash
```

Once the job starts, check the hostname:

```bash
hostname
```

This should show that the shell is running on a compute node rather than the login node.

When finished:

```bash
exit
```

---

# 15. Submit a batch job

Create a Slurm job script:

```bash
nano test.sh
```

Example:

```bash
#!/bin/bash

#SBATCH --job-name=test
#SBATCH --time=00:05:00
#SBATCH --mem=1G
#SBATCH --cpus-per-task=1

echo "Running on:"
hostname

echo "Hello from Hydra!"
```

Submit it:

```bash
sbatch test.sh
```

Check the job:

```bash
squeue -u $USER
```

After completion, Slurm creates an output file similar to:

```text
slurm-123456.out
```

Read it with:

```bash
cat slurm-123456.out
```

---

# 16. Useful first commands after logging in

A useful initial checklist is:

```bash
whoami
hostname
pwd
```

Check storage:

```bash
echo $VSC_HOME
echo $VSC_DATA
echo $VSC_SCRATCH
```

Check available modules:

```bash
module avail
```

Check currently loaded modules:

```bash
module list
```

Check Slurm:

```bash
squeue -u $USER
```

---

# 17. Security

Never commit SSH private keys to GitHub.

The following should **never** be uploaded to GitHub:

```text
~/.ssh/id_rsa_vsc
```

Also never commit:

* SSH passphrases
* Passwords
* API keys
* Access tokens
* Credentials
* `.env` files containing secrets

The public key:

```text
~/.ssh/id_rsa_vsc.pub
```

is not secret, but there is usually no reason to commit it to a project repository.

A good `.gitignore` can include:

```gitignore
# SSH keys
*.pem
id_rsa
id_rsa_*
*.key

# Environment files
.env
.env.*

# Python
__pycache__/
*.py[cod]
.venv/
venv/

# HPC / Slurm
slurm-*.out
```

---

# 18. Final SSH command

Once everything is configured, the normal login command is:

```bash
ssh -i ~/.ssh/id_rsa_vsc vsc11870@login.hpc.vub.be
```

For a cleaner setup, an SSH configuration can be added to:

```text
~/.ssh/config
```

For example:

```sshconfig
Host hydra
    HostName login.hpc.vub.be
    User vsc11870
    IdentityFile ~/.ssh/id_rsa_vsc
    IdentitiesOnly yes
```

Then connecting becomes:

```bash
ssh hydra
```

---

## Summary

The complete workflow is:

```text
Generate SSH key
      ↓
Add passphrase to private key
      ↓
Upload .pub key to VSC account
      ↓
Wait for key propagation
      ↓
SSH to login.hpc.vub.be
      ↓
Authenticate using private key + passphrase
      ↓
Use login node for setup
      ↓
Submit jobs with Slurm
      ↓
Run computations on compute nodes
      ↓
Use VSC_DATA for persistent data
      ↓
Use VSC_SCRATCH for temporary computation data
```

The most important rule is:

> **Keep the private key private, use the public key for VSC authentication, and run heavy computations through Slurm rather than directly on the login node.**
