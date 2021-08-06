1.Create cluster
eksctl create cluster --name=mygitops --region=us-east-1 --zones=us-east-1a,us-east-1b  --without-nodegroup   

eksctl utils associate-iam-oidc-provider \
--region us-east-1 \
--cluster mygitops \
--approve


Create normal Nodegroup 
-----------------------------
eksctl create nodegroup --cluster=mygitops \
--region=us-east-1 \
--name=mygitopsng \
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

export GITHUB_TOKEN=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
export GITHUB_USER=sthitaprajnadas



flux bootstrap github \
--owner=$GITHUB_USER \
--repository=fleet-infra \
--branch=master \
--path=[mygitops] \
--personal

git clone https://github.com/$GITHUB_USER/fleet-infra

cd fleet-infra


flux create source git guestbook-gitops \
--url=https://github.com/sthitaprajnadas/guestbook-gitops \
--branch=master \
--interval=30s \
--export > ./[mygitops]/guestbook-gitops-source.yaml

