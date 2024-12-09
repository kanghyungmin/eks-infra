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
    ```
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
    4) ingressgateway의 status Port 허용 to managedNG SG
    5) Kubernetest Service Port 허용 to managedNG SG

  - Cluster 외부 통신 Ingress 연결 with SSL
    1) Annotation에 다음 필드 추가.
    ```
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 443}]'
    alb.ingress.kubernetes.io/certificate-arn: CA arm ID
    ```
  - Ingress Gateway 분리  
    1) sample-2 예제 참조 
  - CI/CD Pipeline 구성
    1) DockerFile 구성
    ```bash 
    FROM node:22.5.1-alpine

    WORKDIR /app
    
    RUN apk add --no-cache bash
    
    COPY ./src ./src
    COPY ./env ./env
    
    COPY ./package*.json ./
    COPY ./tsconfig*.json ./
    
    RUN yarn install
    
    EXPOSE 3005
    
    CMD ["yarn", "start:dev"]
    # docker build -t dev-api-server -f ./dockerfile.dev ../../
    ```
    2) Dev branch 및 Pro branch 변경 시, CI 구성
    - Dev : PR Merge 시, build 후, ECR 푸쉬
    - Prod : Version Tagging 시, build 후 ECR 푸쉬 
    - Build 후, 각각의 이미지에 대하여 manifest를 저장하고 있는 저장소로 푸쉬  
      (이때, PAT Token 변수 처리 필요)
    ```bash
      - name: Setup Kustomize
        uses: imranismail/setup-kustomize@v1

      - name: Checkout Infra Repository
        uses: actions/checkout@v2
        with:
          repository: Second-Fit/aws-Infra
          ref: master
          token: ${{ secrets.ACTIONS_TOKEN }}
          path: aws-Infra

      - name: Update Kubernetes Resources
        env:
          IMAGE_TAG: ${{ github.sha }}
        run: |
          cd aws-Infra/sample-3/base
          kustomize edit set image ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:${{ env.IMAGE_TAG }}
          git config --global user.email "hmkkang0922@daum.net"
          git config --global user.name "github-actions"
          git commit -am "Update image tag"
          git push
    ```
    3) Argo CD 설치 및 테스트 

  - 모니터링 Stack 구성
  - Pod 기반 HAP
  - Node 기반 AutoScaling
  - 부하 분산에 따른 테스트
  - EKS 기반 AWS Architecuture 수립
    
  - 참조 문서
    1) https://aws.amazon.com/ko/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/
    2) https://devocean.sk.com/blog/techBoardDetail.do?ID=163656
    3) https://aws-diary.tistory.com/60?category=753094
    4) https://whchoi98.gitbook.io/k8s/eks-cicd/github-action-and-argocd-ci-cd
  - 수작업 리스트 
    - DNS A 레코드 추가 후, ALB 연동
    - ALB controller 구성하기
    - SSL 설정 후, CA 맞는 도메인 연결
