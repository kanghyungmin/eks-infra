# Contents    
  - AWS EKS gen by eksctl
    1) [Kubectl 설치, AWS Cli 설치, eksctl cli 설치](https://github.com/kanghyungmin/eks-infra/blob/master/documentation/build_eks.md)
    2) [Cluster 환경 구성](https://github.com/kanghyungmin/eks-infra/blob/master/cluster.yaml)
    3) 실행 명령어
      ```bash
      eksctl create cluster -f cluster.yaml #cluster 생성
      eksctl utils update-cluster-vpc-config --cluster=eks-prod --private-access=true --public-access=true --approve #private-access 설정
      eksctl utils update-cluster-vpc-config --public-access-cidrs= 15.164.94.4/32 --cluster eks-prod --approve # bastion host ip acl
      eksctl delete cluster --name eks-prod --region ap-northeast-2 # cluster 삭제

      # 설정 확인: 서비스 계정과 IAM 역할이 올바르게 연결되었는지 점검합니다.
      kubectl get serviceaccount eks-service-account -n default -o yaml 
      aws eks describe-cluster --name eks-prod --region ap-northeast-2
      ```
  - AWS EKS에 istio 설치 및 설정  
    1) istioctl 설치 
      ```
      $ curl -sL https://istio.io/downloadIstioctl | sh -
      $ cp ~/.istioctl/bin/istioctl ~/bin
      ```
    2) [istioctl 파일 생성](https://github.com/kanghyungmin/eks-infra/blob/master/istio.yaml)
    3) istio 배포 
      ```
      $ istioctl install -f istio.yaml
      ```
    4) istio 배포 확인
      ```
      $ kubectl get pods -n istio-system
      ```
    5) auto injection with namesapce label
      ```
      $ kubectl label namespace default istio-injection=enabled
      $ kubectl label namespace kube-node-lease istio-injection=enabled
      $ kubectl label namespace kube-public istio-injection=enabled
      $ kubectl label namespace kube-system istio-injection=disabled --overwrite
      $ kubectl get ns --show-labels
      ```
    6) istio issue check 
      $ istioctl analyze --all-namespaces --exclude-namespaces kube-system
    7) istio uninstall 
      ```
      $ istioctl uninstall --purge
      ```
  - Istio <> Application Load Balancer 연결
    1) IAM Policy 생성
    ```
      $ curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/install/iam_policy.json
      $ aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
    ```
    2) IAM Role 및 Kubernetes의 Service Account 생성
    ```
      $ eksctl create iamserviceaccount \
              --cluster=eks-prod \
              --namespace=kube-system \
              --name=aws-load-balancer-controller \
              --attach-policy-arn=arn:aws:iam::886436932322:policy/AWSLoadBalancerControllerIAMPolicy \
              --override-existing-serviceaccounts \
              --approve
      $ eksctl get iamserviceaccount --cluster eks-prod # cluster 내 SA 정보 확인 
    ```
    3) Helm 설치 & aws-load-balancer-controller 설치 및 확인
      $ sudo snap install helm --classic
      $ helm repo add eks https://aws.github.io/eks-charts
      $ helm repo update
      $ #aws-load-balancer-controller 설치
      $ helm install aws-load-balancer-controller eks/aws-load-balancer-controller   -n kube-system \
          --set clusterName=eks-pord \
          --set serviceAccount.create=false \
          --set serviceAccount.name=aws-load-balancer-controller
      $ aws-load-balancer-controller가 잘 설치되었는지 확인
      $ kubectl get deployment -n kube-system aws-load-balancer-controller
    ```
    4) ingressgateway의 status port 허용 to managedNG SG
    5) 외부 노드 Service port 허용 to managedNG SG

  - Cluster 외부 통신 Ingress 연결 with SSL
    1) Annotation에 다음 필드 추가.
    ```
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 443}]'
    alb.ingress.kubernetes.io/certificate-arn: CA arm ID
    ```
  - Ingress Gateway 분리
    1) 
  - CI/CD Pipeline 구성
  - 모니터링 Stack 구성(월)
  - Pod 기반 HAP(화)
  - Node 기반 AutoScaling(목)
  - 부하 분산에 따른 테스트
  - EKS 기반 AWS Architecuture 수립
    
  - 참조 문서
    1) https://aws.amazon.com/ko/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/
    2) https://devocean.sk.com/blog/techBoardDetail.do?ID=163656
    3) https://aws-diary.tistory.com/60?category=753094
    4) 
  - 수작업 리스트 
    - DNS A 레코드 추가 후, ALB 연동
    - ALB controller 구성하기
    - SSL 설정 후, CA 맞는 도메인 연결
  - 향후 작업
    - 리소스, namespace, 그리고 external Ingress까지 분리