# 기본 예제 + gitops
# EKS Node에서 개인 ECR에 접근해서 이미지를 가져오는 프로세스
1. AWS CLI로 인증 토큰 생성
    ```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 886436932322.dkr.ecr.ap-northeast-2.amazonaws.com
    ```

2. Kubernetes Secret 생성
    ```
kubectl create secret docker-registry ecr-secret \
  --docker-server=886436932322.dkr.ecr.ap-northeast-2.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region ap-northeast-2) \
  -n dev
    ```

3. Deployment에 Secret 주입
    ```
    spec:
      containers:
      - name: secondfit-api-server-container
        image: 886436932322.dkr.ecr.ap-northeast-2.amazonaws.com/prod/api-server:de4e1c2a4b259fd31632ffdf1aea58d777479363
        ports:
        - containerPort: 3005
      imagePullSecrets:
      - name: ecr-secret
    ```
# argoCD 설치
  - https://devocean.sk.com/blog/techBoardDetail.do?page=&boardType=undefined&query=&ID=163661&searchData=&subIndex=&searchText=&techType=&searchDataSub=&searchDataMain=&comment=
  - 

# Next  
  - 이 예제 + gateway/vs + istio_config + ingress  
    ingressgateway는 2개 한개는 환경, 나머지는 argoCD ingressGateway
  - 다음 dev/prod 환경 구성
  

