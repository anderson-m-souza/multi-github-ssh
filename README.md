# Multiple GitHub accounts using SSH

This a step by step tutorial for my future self, in case I forget how to configure multiple SSH authentications in one machine.

The goal is to use Git in the most practical way, without the need to type funny github addresses while cloning, or adding new remote repositories like `git.github-something.com:account/repository`. Also, I didn't want to manually change the SSH key in the ssh-agent every time I needed to use an inactive account.

Using this configuration, my personal account is used when I'm at any directory inside `~/`, except the ones I put a specific configuration file.

### Install dependencies

- To generate and manage the SSH keys.
openssh
- To copy the SSH key to clipboard.
xclip

### Create and add keys to ssh-agent

Now create a pair of ssh keys for each account. I named my main account keys with the default name, and the additional keys with a postfix to identify them. 

``` bash
$ mkdir ~/.ssh
$ cd ~/.ssh
$ ssh-keygen -t ed25519 -C "me@main.com"
$ id_ed25519
$ ssh-keygen -t ed25519 -C "me@work.com"
$ id_ed25519_work
```

Start the `ssh-agent`.

``` bash
$ eval "$(ssh-agent -s)"
```

Add the private keys to the `ssh-agent`.

``` bash
$ ssh-add ~/.ssh/id_ed25519
$ ssh-add ~/.ssh/id_ed25519_work
```

### Add key to their respective accounts

Copy the content of `~/.ssh/id_ed25519.pub` to clipboard.

``` bash
$ cat id_ed25519.pub | xclip -selection clipboard
```

Log in to the main github account, go to:
Settings > SSH and GPG keys > New SSH key

In the title field type the hostname of the machine, paste the public key into the key field, and add the SSH key.

Copy the content of `~/.ssh/id_ed25519_work.pub` to clipboard.

``` bash
$ cat id_ed25519_work.pub | xclip -selection clipboard
```

Log in to the related github account, go to:
Settings > SSH and GPG keys > New SSH key

In the title field type the same hostname, paste the public key into the key field, and add the SSH key.

### Create gitconfig files

In the home directory, create a file named `.gitconfig` and add these lines.


```
[user]
    name = "Main Name" 
    email = "me@main.com"
[core]
    sshCommand = ssh -i ~/.ssh/id_ed25519 -F /dev/null
[includeIf "gitdir:~/work/"]
    path = ~/work/.gitconfig
[init]
    defaultBranch = main
```

In the secondary directory, create a file named `.gitconfig` and add these lines.


```
[user]
    name = "Work Name"
    email = "me@work.com"
[core]
    sshCommand = ssh -i ~/.ssh/id_ed25519_work -F /dev/null
```

The `sshCommand` part tells Git to load the correct key every time you run Git. And I also set the default branch to `main`.

So if I'm at `~/` or `~/any/directory/`, make some commits, and push a project to Github, git will load the main key, and use the main credentials.

And with this configuration, if I'm at `~/work`, make some commits, push, or clone a project, git will load the work key, and use the work credentials.
