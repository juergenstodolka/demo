# Windows 10: Use multiple SSH keys for remote GitHub repositories

If you have more than one GitHub account, or push to remote repositories using various
accounts, you will need the correct SSH key for each account.

## Starting position

- Operating system: `Windows 10`
- Using `Git Bash`.
- I wanted access from Windows 10 to different repositiories owning by different users.
- I already have an account on GitHub. Let's call the user **`seal-js`**. Working on this account is no problem. On Windows public/privat SSH keys available in directory `~/.ssh`. The public key has beed added to **`seal-js`** GitHub settings. FINE!
- Now I added a new GitHub account for user **`user_2`**. Meanwhile, I created a local repository **`demo`** on my Windows 10 machine. After some commits I wanted to push the local repository to GitHub.

## Create an empty repository for user_2 account

1. Sign in to your GitHub account with `user_2` credentials.
2. Change to `Your Repositories`.
3. Press button `NEW` to create the new empty repository. Call it **`demo`**.

## Create SSH private/public keys for user_2

In detail see article [GitHub] (https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

1. Open `Git Bash`.
2. Change to ssh directory: `cd ~/.ssh`
3. Enter: `ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f "github-user2"`
   Option *-f github-user2* specifies the filename for private/public key. The public key filename ends with `.pub`
4. Repeat step 3. for each GitHub account.
5. Let's go on with `user_2`. Copy content of your public key file `github-user2.pub`.
6. Change to GitHub `user_2` account.
7. Open `Settings` -> `SSH and GPG keys`.
8. Klick on button `NEW SSH key` and paste the public key content into text field. Assign a meaningfiul name and save it.

## Configure SSH on Windows

Now we have tho save our private keys data into ssh config file.

1. Open `Git Bash`.
2. Change to ~/.ssh directory: `cd ~/.ssh`
3. Create empty config file: `touch config`
4. Open the config file with your favourite editor and add all private keys located in directory `~/.ssh`. In our example we have two keys, the first private key used by user `seal-js` is `id_rsa`, the private key of `user_2` is `github-user2`.

### The config file looks like
```
# seal-js account
Host github.com
        HostName github.com
        User seal-js
        IdentityFile ~/.ssh/id_rsa
        IdentitiesOnly yes

# user_2 account
Host github.com-user2
        HostName github.com
        User user_2
        IdentityFile ~/.ssh/github-user2
        IdentitiesOnly yes
```

There are two parts describing the SSH configuration for user `seal-js` and `user_2`. Key `User` is the GitHub user account, `IdentityFile` stores the path to the private key file, `Host` is a symbolic name that should be different from others, `HostName` is self-explanatory. Important is entry `IdentitiesOnlyrun` that has to be set to `yes`. This is needed to get the right private key when connecting to the desired account.


## Add private keys to SSH agent

Now we add the both privat keys to the SSH agent.
```
  ssh-add ~/.ssh/id_rsa           # user seal-js
  ssh-add ~/.ssh/github-user_2    # user user_2
```

Show the added private keys: 
```
$ ssh-add -l
2048 SHA256:hji4ERQ3jlbkK3+xmdC/+5FPi6T3UlAut4tT8/Nvl8k /c/Users/juergen.stodolka/.ssh/id_rsa (RSA)
4096 SHA256:oBLVUkSQyYggQ5MrxmzEZ82UBFiQnUX2q+RSGSiPsaE /c/Users/juergen.stodolka/.ssh/github-user_2 (RSA)
```
### Make this persistent

When you are going to open a new Git Bash all your ssh-agent settings will be lost. To make it persistent save it to `~/.bash_profile`.

The `.bash_profile` is executed after starting a new Git Bash. In our example you have to enter a passphrase for key `github-user_2`.

Here an `.bash_profile` example:

```
  # Add SSH private keys to ssh agent
  # If required, enter a passphrase
  eval `ssh-agent -s`
  ssh-add ~/.ssh/id_rsa
  ssh-add ~/.ssh/github-user_2
```

## Connect local repository *demo* to remote GitHub repository

Arriving at our destination we can connect the local repository `demo` to GitHub remote repository (same name).

```
   git remote add origin git@github.com-user_2:user_2/demo.git
```

Here you can see `github.com-user_2` the value of `Host` stored in `~/.ssh/config`. `user_2` is the GitHub account user and `demo.git` the repository name.

Change to your local repository and use `git remote -v` to display the connection.
```
    origin  git@github.com-user_2:user_2/demo.git (fetch)
    origin  git@github.com-user_2:user_2/demo.git (push)
```

If the connection is wrong, delete it with `git remote remove origin`. Then add the correct connection using `git remote add origin ...`.

The last step: **push local repository to GitHub**
```
    git push -u origin master -- tags
```