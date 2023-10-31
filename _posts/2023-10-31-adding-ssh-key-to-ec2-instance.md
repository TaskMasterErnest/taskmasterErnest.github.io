# Adding an SSH Key to an EC2 Server (Resolving 'Too many authentication failures' issue)

A situation where you provision an EC2 server with the selected key-pair provided (and downloaded by you) but you want to use another SSH key-pair to authenticate into the server ? 

I got you.

A little backstory: 

I am trying to give GitHub Actions an SSH key pair that it can use to connect to and run some commands on a private EC2 server I have provisioned. In order for this to happen, I have to put a public SSH key in the EC2 server and provide the GitHub Actions runner a private key that it can use to connect and authenticate into the server and run my deployment scripts.

I will walk you through the steps:

I am assuming you already have an EC2 server provisioned and its .pem file ready on your local machine.

Here is a twist; you try to login with the command provided by AWS to connect to the instance and you get the error like this:
```bash
$ ssh -i "aws-key.pem" ubuntu@ec2-19-30-68-234.eu-central-2.compute.amazonaws.com
Received disconnect from 19-30-68-234 port 22:2: Too many authentication failures
Disconnected from 19-30-68-234 port 22
```

What this means is that, even when a specific SSH key is to be used in authenticating, the ssh-agent rejects the connection and the SSH client aborts with the above error.

To fix this error, we have to enforce a rule for the ssh-agent to use the specified key to open a connection to the server.

Add this to the `~/.ssh/config` file:
```bash
Host *
      IdentitiesOnly=yes
```
Whether you have an already existing config file ready or not, this will work. If you have config file, you must navigate to the `Host *` portion and add that line to the config there. Save the file.

After this, try authenticating again, It should connect.

Now back to the reson why I am writing this:
1. generate your alternate SSH key you want to use to authenticate into the server.
```bash
ssh-keygen -t ed25519 -f ~/.ssh/mysshkey -C me@local.com
```
2. There should be two new keys in the .ssh file, `mysshkey` and `mysshkey.pub`. The goal here is to move the public key into the EC2 instance's SSH authorized keys. A neat little command to use is this:
```bash
cat mysshkey.pub | ssh -i "aws-key.pem" ubuntu@19-30-68-234 'cat >> .ssh/authorized_keys'
```
The command takes the output of your public key, opens a connection to the EC2 instance and adds the contents to the authorized_keys file in the EC2 instance.
3. Connect to the server with the new SSH key
```bash
ssh -i "mysshkey" ubuntu@19-30-68-234
```

In this extremely short write-up, I have addressed SSH authentication failure issue with a resolution and I have shown how to add an SSH key to an EC2 server.