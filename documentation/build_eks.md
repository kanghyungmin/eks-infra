- Kubectl 설치
  ```bash
    $ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2024-09-12/bin/linux/amd64/kubectl
    $ chmod +x ./kubectl
    $ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
    $ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    $ kubectl version --short --client
  ```
- AWS CLI 설치 & 구성
  ```bash 
    $ sudo apt install unzip
    $ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    $ unzip awscliv2.zip
    $ sudo ./aws/install
    $ aws configure
      AWS Access Key ID [None]: ~~~
      AWS Secret Access Key [None]: ~~~
      Default region name [None]: ap-northeast-2
      Default output format [None]: json

  ```
- eksctl 설치
  ```bash 
    $ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" |  tar xz -C /tmp
    $ sudo mv /tmp/eksctl /usr/local/bin
    $ eksctl version
  ```