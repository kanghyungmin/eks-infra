# Contents    
  - AWS EKS gen by eksctl
    1) [Kubectl 설치, AWS Cli 설치, eksctl cli 설치](https://github.com/kanghyungmin/eks-infra/blob/master/documentation/build_eks.md)
    2) [Cluster 환경 구성](https://github.com/kanghyungmin/eks-infra/blob/master/cluster.yaml)
    3) 실행 명령어
      ```bash
      eksctl create cluster -f cluster.yaml #cluster 생성
      eksctl utils update-cluster-vpc-config --cluster=eks-prod --private-access=true --public-access=true --approve #private-access 설정
      eksctl utils update-cluster-vpc-config --public-access-cidrs=1.1.1.1/32 --cluster eks-prod --approve # bastion host ip acl
      eksctl delete cluster --name eks-prod --region ap-northeast-2 # cluster 삭제
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
      $ istioctl install -f istio-operator.yaml
      ```
    4) istio 배포 확인
      ```
      $ kubectl get pods -n istio-system
      ```
    5) auto injection with namesapce label
      ```
      $ kubectl label namespace default istio-injection=enabled
      ```


  
  - 참조 문서
    1) https://aws.amazon.com/ko/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/
    2) https://devocean.sk.com/blog/techBoardDetail.do?ID=163656
    
    
    

    

     


     kubectl label namespace default istio-injection=enabled

    $ eksctl utils set-public-access-cidrs --cluster=eks-demo 1.1.1.1/32 --approve

    eksctl update addon --name vpc-cni --cluster eks-prod --region ap-northeast-2

    15.164.94.4 => bastion host

    설정 확인: 서비스 계정과 IAM 역할이 올바르게 연결되었는지 점검합니다.
    kubectl get serviceaccount my-service-account -n default -o yaml


- [] (화요일)
     istioctl install -f istio.yaml
- [] Cluster 외부 통신 Ingress 연결(수요일)
- [] Pod와 Container를 위한 HPA(목요일)
- [] AWS EKS의 Karpenter 설정(금요일)
- [] 부하 분산에 따른 테스트
- [] EKS 기반 AWS Architecuture 수립
