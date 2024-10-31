# Lists 
- [] AWS EKS gen by eksctl
     1) Kubectl 설치, AWS Cli 설치, eksctl cli 설치

     $ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2024-09-12/bin/linux/amd64/kubectl
     $ chmod +x ./kubectl
     $ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
     $ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
     $ kubectl version --short --client

     $ sudo apt install unzip
     $ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
       unzip awscliv2.zip
       sudo ./aws/install


    $ aws configure
      AWS Access Key ID [None]: ~~~
      AWS Secret Access Key [None]: ~~~
      Default region name [None]: ap-northeast-2
      Default output format [None]: json

     $ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" |  
       tar xz -C /tmp
     $ sudo mv /tmp/eksctl /usr/local/bin
     $ eksctl version

     eksctl create cluster \
        --version 1.31 \               # Kubernetes 버전 1.21을 사용
        --name eks-demo \              # 클러스터 이름을 eks-demo로 지정
        --vpc-nat-mode HighlyAvailable \ # NAT 게이트웨이를 고가용성(HighlyAvailable) 모드로 설정하여 인터넷 접근성을 높임
        --node-private-networking \    # 워커 노드에 프라이빗 IP만 할당하여, 노드가 프라이빗 서브넷에 위치하도록 함
        --region ap-northeast-2 \      # 클러스터를 ap-northeast-2 (서울) 리전에서 생성
        --node-type t3.medium \        # 워커 노드의 인스턴스 타입을 t3.medium으로 설정
        --nodes 2 \                    # 워커 노드를 2개 생성
        --with-oidc \                  # IAM 역할을 Kubernetes에서 사용할 수 있도록 OIDC를 활성화
        --ssh-access \                 # SSH 접근을 활성화
        --ssh-public-key thomas \      # SSH 접근을 위해 'thomas'라는 이름의 공개 키를 사용
        --managed                      # AWS에서 관리되는 노드 그룹을 생성하여 자동 업데이트와 패치 적용을 받음

    eksctl delete cluster --name eks-demo --region ap-northeast-2

- [] AWS EKS에 lstio 설치 및 설정
- [] Cluster 외부 통신 Ingress 연결
- [] Pod와 Container를 위한 HPA
- [] AWS EKS의 Karpenter 설정
- [] 부하 분산에 따른 테스트
