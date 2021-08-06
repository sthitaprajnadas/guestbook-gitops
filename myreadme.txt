1.Create cluster
eksctl create cluster --name=mycluster --region=us-east-1 --zones=us-east-1a,us-east-1b  --without-nodegroup   

eksctl utils associate-iam-oidc-provider \
--region us-east-1 \
--cluster mycluster \
--approve


Create normal Nodegroup 
-----------------------------
eksctl create nodegroup --cluster=mycluster \
--region=us-east-1 \
--name=myclustersng \
--node-type=t3.medium \
--nodes=2 \
--nodes-min=2 \
--nodes-max=4 \
--node-volume-size=20 \
--ssh-access \
--ssh-public-key=kube-demo-east1 \
--managed \
--asg-access \
--external-dns-access \
--full-ecr-access \
--appmesh-access \
--alb-ingress-access

 eksctl delete nodegroup eksdemo1-ng-public1  --cluster=myclusters
 eksctl delete cluster --name=mycluster

 ================================================

export GITHUB_TOKEN=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
export GITHUB_USER=sthitaprajnadas
============================================


aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 437637487786.dkr.ecr.us-east-1.amazonaws.com




flux bootstrap github \
--owner=$GITHUB_USER \
--repository=guest-infra \
--branch=master \
--path=[mycluster] \
--personal

git clone https://github.com/$GITHUB_USER/guest-infra

cd guest-infra


flux create source git guestbook-gitops \
--url=https://github.com/sthitaprajnadas/guestbook-gitops \
--branch=master \
--interval=30s \
--export > ./[mycluster]/guestbook-gitops-source.yaml


Configure a flux kustomization

flux create kustomization guestbook-gitops \
--source=guestbook-gitops \
--path="./deploy" \
--prune=true \
--validation=client \
--interval=1h \
--export > ./[mycluster]/guestbook-gitops-sync.yaml


git add -A && git commit -m "add guestbook-gitops deploy" && git push

watch flux get kustomizations



Uninstall flux:
  flux uninstall --namespace=flux-system


