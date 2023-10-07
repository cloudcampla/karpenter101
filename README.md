# Instalación KARPENTER


Con el sigueinte manifiesto de CloudFormation vamos a crear los recursos necesarios para aprovisionar Karpenter en nuestro cluster:

````
curl -fsSL https://raw.githubusercontent.com/aws/karpenter/v0.31.0/website/content/en/preview/getting-started/getting-started-with-karpenter/cloudformation.yaml > $(pwd)/temp.yaml
&& aws cloudformation deploy
--stack-name "Karpenter-agosto-eks-cluster"
--template-file "$(pwd)/temp.yaml"
--capabilities CAPABILITY_NAMED_IAM
--parameter-overrides "ClusterName=agosto-eks-cluster"
````

Vamos a encontrar que se crearon 2 elementos especialmente importantes


  - KarpenterNodeRole-agosto-eks-cluster: 
      
        ARN IAM ROLE: arn:aws:iam::628924030472:role/KarpenterNodeRole-agosto-eks-cluster
    
  - KarpenterControllerPolicy-agosto-eks-cluster: 
      
        ARN IAM POLICY: arn:aws:iam::628924030472:policy/KarpenterControllerPolicy-agosto-eks-cluster


2. Creamos un role con IRSA y le asignamos la política creada anteriormente
   
```
eksctl create iamserviceaccount \
--cluster=agosto-eks-cluster \
--namespace=karpenter \
--name=karpenter \
--role-name=agosto-eks-cluster-karpenter-role \
--attach-policy-arn "arn:aws:iam::628924030472:policy/KarpenterControllerPolicy-agosto-eks-cluster" \
--role-only \
--approve
```

3. Añadimos la siguiente tag, a todas las subnets y grupos de seguridad en las que actualmente esperamos aprovisionar los recursos del cluster:

````
     Key: karpenter.sh/discovery

     Value: agosto-eks-cluster
````
    Nota: También podemos crear un nuevo Security Group para hacer pruebas asignandole la tag anterior.
   

4. Editamos el configmap aws-auth :

`kubectl edit configmap aws-auth -n kube-system`

y añadimos:

````
- groups:
  - system:bootstrappers
  - system:nodes
  rolearn: arn:aws:iam::628924030472:role/KarpenterNodeRole-agosto-eks-cluster
  username: system:node:{{EC2PrivateDNSName}}
````

5. Desplegamos Karpenter con FluxCD como hicimos en esta parte del laboratorio:

   https://github.com/cloudcampla/iac-fluxcd-demo/tree/main/infrastructure/demo/karpenter
