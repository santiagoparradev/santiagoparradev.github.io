---
title: Set SSH Key to be used automatically
date: 2024-11-20 12:00:00 +0000
layout: post
categories: [git]
tags: [git]
description: "short solution when dealing with ssh keys"
---

## 🚩 **The Problem**

You’ve gone through the process of creating an SSH key for authentication, but now you want to use it with your Git repositories without hassle.

---

## 🛠️ **Steps to Configure Your SSH Key**

### Step 1: Generate an SSH Key and Add It to the SSH Agent  
Follow the official guide to generate and add your SSH key:  
[Generate SSH Key (GitHub Guide)](https://docs.github.com/en/authentication/connecting-to-github-with-ssh).

---

### Step 2: Configure Your SSH `config` File  

Update your SSH configuration file (`~/.ssh/config`) to associate specific keys with different hosts.  
Here’s an example:

```plaintext
# Key for work GitHub
Host work.github.com
    User git
    HostName github.com
    IdentityFile /c/Users/santi/.ssh/santiago@workgmail.com/id_ed25519
    IdentitiesOnly yes

# Key for personal GitHub
Host github.com
    User git
    HostName github.com
    IdentityFile /c/Users/santi/.ssh/santiagoparradev@gmail.com/id_ed25519
    AddKeysToAgent yes
    IdentitiesOnly yes
```
**Notes**: Your alias goes after Host for instance `work.github.com` OR `github.com`

---

### Step 3: Set Up Git to Use Your SSH Key  

#### 🎯 **Add a Remote with SSH**  
If you’ve initialized a repository but haven’t set up the remote yet, use this format:  
```bash
git remote add origin git@{alias}:{username}/{repository}.git
```

Examples:  
```bash
git remote add origin git@work.github.com:santiagoparradevwork/my-work-repository.git
git remote add origin git@github.com:santiagoparradev/my-repository.git
```

#### 🎯 **Clone a Repository with SSH**  
If you want to clone a repository directly:  
```bash
git clone git@{alias}:{username}/{repository}.git
```

Examples:  
```bash
git clone git@work.github.com:santiagoparradevwork/my-work-repository.git
git clone git@github.com:santiagoparradev/my-repository.git
```

---

### Step 4: Specify a Custom SSH Key (Optional)

If you need to use a specific SSH key for a repository, configure Git to use it:  
```bash
git config core.sshCommand "ssh -i ~/.ssh/your_private_key"
```

---

### 💡 **Important Notes**

- You only need to configure the SSH key **once per repository**.  
- After setup, Git will automatically use the correct SSH key for the configured host or repository.

---

## 🚀 **Now You’re Ready!**  
You can safely use your Git commands, and Git will handle the correct SSH key automatically. Examples:  
```bash
git push -u origin master
git clone git@github.com:username/repository.git
```


## 🚀 **Stick to the basics**  
Accep sometimes you might just end up doing 
```bash
eval $(ssh-agent -s)
ssh-add /c/Users/santi/.ssh/santiagoparradev@gmail.com/id_ed25519
git clone git@github.com:santiagoparradev/my-repository.git
```

🎉 Enjoy seamless Git operations with your SSH keys configured!
