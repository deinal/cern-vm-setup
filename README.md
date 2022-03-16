# vm-setup

Set up a virtual machine for your Kubeflow projects

## Create a virtual machine

This part is well documented by the cloud team. You can create a vm through the
- browser: https://clouddocs.web.cern.ch/tutorial_using_a_browser/index.html
- cli https://clouddocs.web.cern.ch/tutorial/index.html

I chose to create a machine named `jec-vm` with CentOS 8 which is reflected in the setup commands below.

At this point you should be able to see the vm by executing the following on lxplus8

```
openstack server show jec-vm
```

## Setup

Begin by entering your vm
```
ssh root@jec-vm
```

### User

I add my own user to the machine
```
sudo useradd -G wheel dholmber
```

Set a password for my user
```
sudo passwd dholmber
```

### SSH

Add public lxplus key to your user's authorized keys
```
mkdir /home/dholmber/.ssh
chown dholmber /home/dholmber/.ssh
cp .ssh/authorized_keys /home/dholmber/.ssh/
chown dholmber /home/dholmber/.ssh/authorized_keys
exit
```

Add your vm to your local `.ssh/config` file and append your local public key to `authorized_keys` in your vm allowing you to jump directly to it from your local machine.

```
Host lxplus8
    User dholmber
    HostName lxplus8.cern.ch
    GSSAPIDelegateCredentials yes
    GSSAPIAuthentication yes
    GSSAPITrustDns yes

Host jec-vm
    User dholmber
    HostName jec-vm
    ProxyJump lxplus8
```

Re-enter the vm as your user
```
ssh jec-vm
```

### Install packages

Git, naturally
```
sudo yum install -y git
git config --global user.name "dholmber"
git config --global user.email "daniel.holmberg@cern.ch"
```

Spice up the shell, highly optional
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
```

Docker
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo groupadd docker
sudo usermod -aG docker $USER
exit
ssh jec-vm
docker run hello-world
```

Python3 and the Kubeflow pipelines SDK
```
sudo yum install -y python3
pip3 install --upgrade --user kfp
```

SSO cookie generator
```
sudo yum install -y gcc python3-devel krb5-devel
git clone https://gitlab.cern.ch/authzsvc/tools/auth-get-sso-cookie.git
cd auth-get-sso-cookie
python3 setup.py install --user
```

## Get to work

Now you should be good to start working on Kubeflow projects from your directly from your vm.

```
git clone https://gitlab.cern.ch/dholmber/jec-pipeline.git
cd jec-pipeline
auth-get-sso-cookie -u https://ml.cern.ch -o cookies.txt
python3 pipeline.py
```

Docker is heavily used when developing Kubeflow pipelines, and in the vm you can iterate changes quickly by pushing to the CERN registry for example.
```
docker login registry.cern.ch
docker build training -t registry.cern.ch/ml/jec-training
docker push registry.cern.ch/ml/jec-training
```
