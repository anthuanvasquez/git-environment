# Git Setup for Multiple Environments on MacOS

## Overview

This repository contains configurations for setting up a Git environment on MacOS using SSH and GPG keys. The configurations are tailored to different working directories where the Git settings change accordingly. This documentation will guide you through the process of generating and configuring SSH and GPG keys from scratch, and explain how to use the provided `.gitconfig` files and SSH configuration for a seamless development setup.

## Setting Up SSH Keys

### Generate a New SSH Key

To generate a new SSH key, open your terminal and use the following command:

```bash
ssh-keygen -t ed25519 -C "username@example.com"
```

If you are using a legacy system that doesn't support the `Ed25519` algorithm, use:

```bash
ssh-keygen -t rsa -b 4096 -C "username@example.com"
```

Follow the prompts to save the key to the default location (`~/.ssh/id_ed25519` for Ed25519 keys). When prompted, you can set a passphrase for additional security.

### Adding SSH Key to the SSH Agent

Start the SSH agent in the background:

```bash
eval "$(ssh-agent -s)"
```

Add your SSH private key to the SSH agent:

```bash
ssh-add ~/.ssh/id_ed25519
```

For legacy systems:

```bash
ssh-add ~/.ssh/id_rsa
```

## Setting Up GPG Keys

### Generate a New GPG Key

To generate a new GPG key, use the following command:

```bash
gpg --full-generate-key
```

Follow the prompts to configure your GPG key. Choose the following options when prompted:
- **Kind of key**: RSA and RSA
- **Key size**: 4096 bits
- **Key expiration**: Choose a suitable expiration period or leave it as "0" for no expiration
- **Real name**: Your Name
- **Email address**: username@example.com
- **Comment**: Optional
- **Passphrase**: Set a secure passphrase

After generating the key, list your keys to find the key ID:

```bash
gpg --list-secret-keys --keyid-format LONG
```

Export the key:

```bash
gpg --armor --export your-key-id
```

## Configuring Git

### Main `.gitconfig` File

Create the main `.gitconfig` file in your home directory (`~/.gitconfig`) with the following content:

```ini
[user]
  name = Your Name
  email = username@domain.com
  signingkey = your-public-ssh-key-string

[commit]
  gpgsign = true

[gpg]
  format = ssh

[gpg "ssh"]
  program = /Applications/1Password.app/Contents/MacOS/op-ssh-sign

[includeIf "gitdir:~/Sites/acme/"]
  path = ~/Sites/acme/.gitconfig

[includeIf "gitdir:~/Sites/alpha/"]
  path = ~/Sites/alpha/.gitconfig
```

### Directory-Specific `.gitconfig` Files

For the `acme` directory (`~/Sites/acme/.gitconfig`):

```ini
[user]
  email = username@acme.com
  signingkey = your-gpg-key-string

[gpg]
  format = openpgp
```

For the `alpha` directory (`~/Sites/alpha/.gitconfig`):

```ini
[user]
  email = username@alpha.com
  signingkey = your-gpg-key-string

[gpg]
  format = openpgp
```

### SSH Configuration

Create an SSH config file (`~/.ssh/config`) with the following content:

```plaintext
# Personal Use
Host github.com
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/your_id_rsa_private_key

# Acme
Host github_acme
    HostName github.com
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/your_acme_id_rsa_private_key

# Alpha
Host github_alpha
    HostName github.com
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/your_alpha_id_rsa_private_key
```

## Usage Instructions

1. **Clone the repository**: Use your preferred method to clone the repository that contains this configuration setup.

2. **Customize `.gitconfig` and SSH configurations**: Edit the `.gitconfig` and SSH config files with your own details. Replace placeholders like `your-public-ssh-key-string`, `your-gpg-key-string`, and paths with your actual information.

3. **Work with different directories**: The provided configuration automatically applies different Git settings based on the working directory (`acme` and `alpha`). Ensure your directory structure matches the configuration (e.g., `~/Sites/acme` and `~/Sites/alpha`).

4. **Commit Signing**: Your commits will be signed using the specified GPG key. Ensure your GPG key is added to your GitHub account by following [GitHub's guide](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account).

5. **SSH Key Usage**: Use the appropriate SSH key for different repositories by configuring your remote URLs with the corresponding host (`github_acme` or `github_alpha`).
