# 기본 예제 + gitops
# Next  
  - 이 예제 + gateway/vs + istio_config + ingress  
    ingressgateway는 2개 한개는 환경, 나머지는 argoCD ingressGateway
  - 다음 dev/prod 환경 구성
  

1. AWS CLI로 인증 토큰 생성
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 886436932322.dkr.ecr.ap-northeast-2.amazonaws.com

2. Kubernetes Secret 생성
kubectl create secret docker-registry ecr-secret \
  --docker-server=886436932322.dkr.ecr.ap-northeast-2.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region ap-northeast-2) \
  -n dev

3. Deployment에 Secret 주입
  spec:
  containers:
    - name: api-server
      image: <ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/<IMAGE_NAME>:<TAG>
  imagePullSecrets:
    - name: ecr-secret