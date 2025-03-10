# crossplane-eks-cluster
[![test-composition-and-publish-to-ghcr](https://github.com/jonashackt/crossplane-eks-cluster/actions/workflows/test-composition-and-publish-to-ghcr.yml/badge.svg)](https://github.com/jonashackt/crossplane-eks-cluster/actions/workflows/test-composition-and-publish-to-ghcr.yml)
![crossplane-version](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fraw.githubusercontent.com%2Fjonashackt%2Fcrossplane-eks-cluster%2Fmain%2Fcrossplane%2Finstall%2FChart.yaml&query=%24.dependencies%5B%3A1%5D.version&label=crossplane&color=blue)
![provider-aws-ec2](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fraw.githubusercontent.com%2Fjonashackt%2Fcrossplane-eks-cluster%2Fmain%2Fcrossplane%2Fprovider%2Fupbound-provider-aws-ec2.yaml&query=%24.spec.package&label=provider-aws-ec2&color=rgb(109%2C%20100%2C%20245))
![provider-aws-eks](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fraw.githubusercontent.com%2Fjonashackt%2Fcrossplane-eks-cluster%2Fmain%2Fcrossplane%2Fprovider%2Fupbound-provider-aws-eks.yaml&query=%24.spec.package&label=provider-aws-eks&color=rgb(109%2C%20100%2C%20245))
![provider-aws-iam](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fraw.githubusercontent.com%2Fjonashackt%2Fcrossplane-eks-cluster%2Fmain%2Fcrossplane%2Fprovider%2Fupbound-provider-aws-iam.yaml&query=%24.spec.package&label=provider-aws-iam&color=rgb(109%2C%20100%2C%20245))
[![License](http://img.shields.io/:license-mit-blue.svg)](https://github.com/jonashackt/crossplane-eks-cluster/blob/master/LICENSE)
[![renovateenabled](https://img.shields.io/badge/renovate-enabled-yellow)](https://renovatebot.com)

Crossplane Configuration delivering CRDs to provision AWS EKS clusters

This set of Crossplane (Nested) Compositions to provision a AWS EKS cluster was originally started in https://github.com/jonashackt/crossplane-argocd - but then scaled out to a separate repository using Crossplane's [Configuration Package feature](https://docs.crossplane.io/latest/concepts/packages/) to build a OCI image from the CRDs in this repo. 

There's not really much documentation about Nested Compositions. There's [this section in the upbound docs about "Layering composite resources"](https://docs.upbound.io/xp-arch-framework/building-apis/building-apis-compositions/#layering-composite-resources).

Only some hints [like this about the role of the `XRD.status` field](https://docs.upbound.io/xp-arch-framework/building-apis/building-apis-xrds/#xrd-status).

Most information [is provided by this blog post](https://vrelevant.net/crossplane-beyond-the-basics-nested-xrs-and-composition-selectors/) and some examples like [this (watch out, this is based on the crossplane aws provider!)](https://github.com/cem-altuner/crossplane-prod-ready-eks) and [this](https://github.com/upbound/configuration-eks).

# How to use it

As described in https://docs.crossplane.io/latest/concepts/packages/#install-a-configuration use a manifest like the following to install the Configuration:

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Configuration
metadata:
  name: crossplane-eks-cluster
spec:
  package: ghcr.io/jonashackt/crossplane-eks-cluster:v0.0.2
```

`apply -f` the file and create a Claim like shown in the `examples` folder at []`examples/claim.yaml`](examples/claim.yaml):

```yaml
apiVersion: k8s.crossplane.jonashackt.io/v1alpha1
kind: KubernetesCluster
metadata:
  namespace: default
  name: deploy-target-eks
spec:
  id: deploy-target-eks
  parameters:
    region: eu-central-1
    nodes:
      count: 3
  writeConnectionSecretToRef:
    name: eks-cluster-kubeconfig
```

Imagine a cluster (like the one bootstrapped in https://github.com/jonashackt/crossplane-argocd) and run `kubectl get crossplane`:

```shell
$ kubectl get crossplane
NAME                                    AGE
providerconfig.aws.upbound.io/default   7d3h

NAME                                                                          HEALTHY   REVISION   IMAGE                                                STATE    DEP-FOUND   DEP-INSTALLED   AGE
providerrevision.pkg.crossplane.io/provider-aws-ec2-150095bdd614              True      1          xpkg.upbound.io/upbound/provider-aws-ec2:v1.1.1      Active   1           1               7d3h
providerrevision.pkg.crossplane.io/provider-aws-eks-fbb6768e46c0              True      1          xpkg.upbound.io/upbound/provider-aws-eks:v1.1.1      Active   1           1               7d3h
providerrevision.pkg.crossplane.io/provider-aws-iam-9565c6312cd0              True      1          xpkg.upbound.io/upbound/provider-aws-iam:v1.1.1      Active   1           1               7d3h
providerrevision.pkg.crossplane.io/upbound-provider-family-aws-11fe5ecef831   True      1          xpkg.upbound.io/upbound/provider-family-aws:v1.1.1   Active                               7d4h

NAME                                                     INSTALLED   HEALTHY   PACKAGE                                              AGE
provider.pkg.crossplane.io/provider-aws-ec2              True        True      xpkg.upbound.io/upbound/provider-aws-ec2:v1.1.1      7d3h
provider.pkg.crossplane.io/provider-aws-eks              True        True      xpkg.upbound.io/upbound/provider-aws-eks:v1.1.1      7d3h
provider.pkg.crossplane.io/provider-aws-iam              True        True      xpkg.upbound.io/upbound/provider-aws-iam:v1.1.1      7d3h
provider.pkg.crossplane.io/upbound-provider-family-aws   True        True      xpkg.upbound.io/upbound/provider-family-aws:v1.1.1   7d4h

NAME                                                AGE
deploymentruntimeconfig.pkg.crossplane.io/default   7d4h

NAME                                        AGE    TYPE         DEFAULT-SCOPE
storeconfig.secrets.crossplane.io/default   7d4h   Kubernetes   crossplane-system
```

Now after applying this Configuration here, there should appear all the Compositions and XRDs:

```shell
$ kubectl get crossplane
NAME                                                                                                       ESTABLISHED   OFFERED   AGE
compositeresourcedefinition.apiextensions.crossplane.io/xeksclusters.eks.aws.crossplane.jonashackt.io      True          True      4s
compositeresourcedefinition.apiextensions.crossplane.io/xkubernetesclusters.k8s.crossplane.jonashackt.io   True          True      4s
compositeresourcedefinition.apiextensions.crossplane.io/xnetworkings.net.aws.crossplane.jonashackt.io      True          True      4s

NAME                                                                         REVISION   XR-KIND              XR-APIVERSION                               AGE
compositionrevision.apiextensions.crossplane.io/aws-eks-4c2092f              1          XEKSCluster          eks.aws.crossplane.jonashackt.io/v1alpha1   4s
compositionrevision.apiextensions.crossplane.io/kubernetes-cluster-2b6e754   1          XKubernetesCluster   k8s.crossplane.jonashackt.io/v1alpha1       4s
compositionrevision.apiextensions.crossplane.io/networking-3869153           1          XNetworking          net.aws.crossplane.jonashackt.io/v1alpha1   4s

NAME                                                         XR-KIND              XR-APIVERSION                               AGE
composition.apiextensions.crossplane.io/aws-eks              XEKSCluster          eks.aws.crossplane.jonashackt.io/v1alpha1   4s
composition.apiextensions.crossplane.io/kubernetes-cluster   XKubernetesCluster   k8s.crossplane.jonashackt.io/v1alpha1       4s
composition.apiextensions.crossplane.io/networking           XNetworking          net.aws.crossplane.jonashackt.io/v1alpha1   4s

NAME                                    AGE
providerconfig.aws.upbound.io/default   7d3h

NAME                                                                          HEALTHY   REVISION   IMAGE                                              STATE    DEP-FOUND   DEP-INSTALLED   AGE
configurationrevision.pkg.crossplane.io/crossplane-eks-cluster-edf0e1ba1b0c   True      1          ghcr.io/jonashackt/crossplane-eks-cluster:v0.0.2   Active   4           4               6s

NAME                                                     INSTALLED   HEALTHY   PACKAGE                                            AGE
configuration.pkg.crossplane.io/crossplane-eks-cluster   True        True      ghcr.io/jonashackt/crossplane-eks-cluster:v0.0.2   6s

NAME                                                                          HEALTHY   REVISION   IMAGE                                                STATE    DEP-FOUND   DEP-INSTALLED   AGE
providerrevision.pkg.crossplane.io/provider-aws-ec2-150095bdd614              True      1          xpkg.upbound.io/upbound/provider-aws-ec2:v1.1.1      Active   1           1               7d3h
providerrevision.pkg.crossplane.io/provider-aws-eks-fbb6768e46c0              True      1          xpkg.upbound.io/upbound/provider-aws-eks:v1.1.1      Active   1           1               7d3h
providerrevision.pkg.crossplane.io/provider-aws-iam-9565c6312cd0              True      1          xpkg.upbound.io/upbound/provider-aws-iam:v1.1.1      Active   1           1               7d3h
providerrevision.pkg.crossplane.io/upbound-provider-family-aws-11fe5ecef831   True      1          xpkg.upbound.io/upbound/provider-family-aws:v1.1.1   Active                               7d4h

NAME                                                     INSTALLED   HEALTHY   PACKAGE                                              AGE
provider.pkg.crossplane.io/provider-aws-ec2              True        True      xpkg.upbound.io/upbound/provider-aws-ec2:v1.1.1      7d3h
provider.pkg.crossplane.io/provider-aws-eks              True        True      xpkg.upbound.io/upbound/provider-aws-eks:v1.1.1      7d3h
provider.pkg.crossplane.io/provider-aws-iam              True        True      xpkg.upbound.io/upbound/provider-aws-iam:v1.1.1      7d3h
provider.pkg.crossplane.io/upbound-provider-family-aws   True        True      xpkg.upbound.io/upbound/provider-family-aws:v1.1.1   7d4h

NAME                                                AGE
deploymentruntimeconfig.pkg.crossplane.io/default   7d4h

NAME                                        AGE    TYPE         DEFAULT-SCOPE
storeconfig.secrets.crossplane.io/default   7d4h   Kubernetes   crossplane-system
```


# Building a Nested Composition for EKS with Crossplane

## Bootstrap a EKS cluster with Crossplane

https://marketplace.upbound.io/providers/upbound/provider-aws-eks/

Inspiration taken from https://github.com/cem-altuner/crossplane-prod-ready-eks (watch out, this is based on the crossplane aws provider!), https://github.com/upbound/configuration-eks 


### Add EKS, ECS & IAM Providers

We first need to add 3 more Crossplane Providers from the upbound provider families: `provider-aws-eks` and `provider-aws-ec2`.

[`upbound/provider-aws/provider/provider-aws-eks.yaml`](upbound/provider-aws/provider/provider-aws-eks.yaml):

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: upbound-provider-aws-eks
spec:
  package: xpkg.upbound.io/upbound/provider-aws-eks:v1.2.1
  packagePullPolicy: IfNotPresent # Only download the package if it isn’t in the cache.
  revisionActivationPolicy: Automatic # Otherwise our Provider never gets activate & healthy
  revisionHistoryLimit: 1
```

the [`upbound/provider-aws/provider/provider-aws-ec2.yaml`](upbound/provider-aws/provider/provider-aws-ec2.yaml):

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: upbound-provider-aws-ec2
spec:
  package: xpkg.upbound.io/upbound/provider-aws-ec2:v1.2.1
  packagePullPolicy: IfNotPresent # Only download the package if it isn’t in the cache.
  revisionActivationPolicy: Automatic # Otherwise our Provider never gets activate & healthy
  revisionHistoryLimit: 1
```

and the [`crossplane/provider/upbound-provider-aws-iam.yaml`](crossplane/provider/upbound-provider-aws-iam.yaml):

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: upbound-provider-aws-iam
spec:
  package: xpkg.upbound.io/upbound/provider-aws-iam:v1.2.1
  packagePullPolicy: IfNotPresent # Only download the package if it isn’t in the cache.
  revisionActivationPolicy: Automatic # Otherwise our Provider never gets activate & healthy
  revisionHistoryLimit: 1
```

You can also use the great Crossplane CLI command `crossplane beta trace` to see all the resources in the scope of your Claim:

```shell
crossplane beta trace kubernetesclusters.k8s.crossplane.jonashackt.io/deploy-target-eks -o wide 
NAME                                                               RESOURCE                                SYNCED   READY   STATUS
KubernetesCluster/deploy-target-eks (default)                                                              True     True    Available
└─ XKubernetesCluster/deploy-target-eks-chzjb                                                              True     True    Available
   ├─ XNetworking/deploy-target-eks-chzjb-7zw9c                    compositeNetworkEKS                     True     True    Available
   │  ├─ VPC/deploy-target-eks                                     platform-vcp                            True     True    Available
   │  ├─ InternetGateway/deploy-target-eks-chzjb-n2gx9             gateway                                 True     True    Available
   │  ├─ Subnet/deploy-target-eks-chzjb-fdhpp                      subnet-public-eu-central-1a             True     True    Available
   │  ├─ Subnet/deploy-target-eks-chzjb-tt4pb                      subnet-public-eu-central-1b             True     True    Available
   │  ├─ Subnet/deploy-target-eks-chzjb-crx5m                      subnet-public-eu-central-1c             True     True    Available
   │  ├─ SecurityGroup/deploy-target-eks                           securitygroup-cluster                   True     True    Available
   │  ├─ SecurityGroupRule/deploy-target-eks-chzjb-8wlkv           securitygrouprule-cluster-inbound       True     True    Available
   │  ├─ SecurityGroupRule/deploy-target-eks-chzjb-tjtxz           securitygrouprule-cluster-outbound      True     True    Available
   │  ├─ Route/deploy-target-eks-chzjb-wh7gl                       route                                   True     True    Available
   │  ├─ RouteTable/deploy-target-eks-chzjb-wc5lh                  routeTable                              True     True    Available
   │  ├─ MainRouteTableAssociation/deploy-target-eks-chzjb-xsgss   mainRouteTableAssociation               True     True    Available
   │  ├─ RouteTableAssociation/deploy-target-eks-chzjb-9gt7h       RouteTableAssociation-public-a          True     True    Available
   │  ├─ RouteTableAssociation/deploy-target-eks-chzjb-px4g5       RouteTableAssociation-public-b          True     True    Available
   │  └─ RouteTableAssociation/deploy-target-eks-chzjb-mzlqh       RouteTableAssociation-public-c          True     True    Available
   └─ XEKSCluster/deploy-target-eks-chzjb-fcmp7                    compositeClusterEKS                     True     True    Available
      ├─ Cluster/deploy-target-eks                                 eksCluster                              True     True    Available
      ├─ ClusterAuth/deploy-target-eks-chzjb-4hq4d                 kubernetesClusterAuth                   True     True    Available
      ├─ Role/deploy-target-eks-chzjb-mdmx9                        clusterRole                             True     True    Available
      ├─ RolePolicyAttachment/deploy-target-eks-chzjb-wwvws        clusterRolePolicyAttachment             True     True    Available
      ├─ NodeGroup/deploy-target-eks-chzjb-zt66c                   nodeGroupPublic                         True     True    Available
      ├─ Role/deploy-target-eks-chzjb-ccvgp                        nodegroupRole                           True     True    Available
      ├─ RolePolicyAttachment/deploy-target-eks-chzjb-bbgrr        workerNodeRolePolicyAttachment          True     True    Available
      ├─ RolePolicyAttachment/deploy-target-eks-chzjb-nk2m4        cniRolePolicyAttachment                 True     True    Available
      └─ RolePolicyAttachment/deploy-target-eks-chzjb-tdr2b        containerRegistryRolePolicyAttachment   True     True    Available
```


### The EC2 Networking Composition

Can be found in `apis/networking/` directory:

* XRD: [`apis/networking/definition.yaml`](apis/networking/definition.yaml)


<details>
  <summary>expand full yaml</summary>

  ```yaml
  apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xnetworkings.net.aws.crossplane.jonashackt.io
spec:
  group: net.aws.crossplane.jonashackt.io
  names:
    kind: XNetworking
    plural: xnetworkings
  claimNames:
    kind: Networking
    plural: networkings
  versions:
    - name: v1alpha1
      served: true
      referenceable: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            # defining input parameters
            spec:
              type: object
              properties:
                id:
                  type: string
                  description: ID of this Network that other objects will use to refer to it.
                parameters:
                  type: object
                  description: Network configuration parameters.
                  properties:
                    region:
                      type: string
                  required:
                    - region
              required:
                - id
                - parameters
            # defining return values
            status:
              type: object
              properties:
                subnetIds:
                  type: array
                  items:
                    type: string
                securityGroupClusterIds:
                  type: array
                  items:
                    type: string
  ```
</details>

* Composition: [`apis/networking/composition.yaml`](apis/networking/composition.yaml)

<details>
  <summary>expand full yaml</summary>
  
  ```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: networking
  labels:
    provider: aws
spec:
  compositeTypeRef:
    apiVersion: net.aws.crossplane.jonashackt.io/v1alpha1
    kind: XNetworking

  writeConnectionSecretsToNamespace: crossplane-system

  patchSets:
  - name: networkconfig
    patches:
    - type: FromCompositeFieldPath
      fromFieldPath: spec.id
      toFieldPath: metadata.labels[net.aws.crossplane.jonashackt.io/network-id] # the network-id other Composition MRs (like EKSCluster) will use
    - type: FromCompositeFieldPath
      fromFieldPath: spec.parameters.region
      toFieldPath: spec.forProvider.region

  resources:
    ### VPC and InternetGateway
    - name: platform-vcp
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: VPC
        spec:
          forProvider:
            cidrBlock: 10.0.0.0/16
            enableDnsSupport: true
            enableDnsHostnames: true
            tags:
              Owner: Platform Team
              Name: platform-vpc
      patches:
        - type: PatchSet
          patchSetName: networkconfig
        - fromFieldPath: spec.id
          toFieldPath: metadata.name
    
    - name: gateway
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: InternetGateway
        spec:
          forProvider:
            vpcIdSelector:
              matchControllerRef: true
      patches:
        - type: PatchSet
          patchSetName: networkconfig


    ### Subnet Configuration
    - name: subnet-public-eu-central-1a
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: Subnet
        metadata:
          labels:
            access: public
        spec:
          forProvider:
            mapPublicIpOnLaunch: true
            cidrBlock: 10.0.0.0/24
            vpcIdSelector:
              matchControllerRef: true
            tags:
              kubernetes.io/role/elb: "1"
      patches:
        - type: PatchSet
          patchSetName: networkconfig
        # define eu-central-1a as zone & availabilityZone
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: metadata.labels.zone
          transforms:
            - type: string
              string:
                fmt: "%sa"
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: spec.forProvider.availabilityZone
          transforms:
            - type: string
              string:
                fmt: "%sa"
        # provide the subnetId for later use as status.subnetIds entry
        - type: ToCompositeFieldPath
          fromFieldPath: metadata.annotations[crossplane.io/external-name]
          toFieldPath: status.subnetIds[0]
    
    - name: subnet-public-eu-central-1b
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: Subnet
        metadata:
          labels:
            access: public
        spec:
          forProvider:
            mapPublicIpOnLaunch: true
            cidrBlock: 10.0.1.0/24
            vpcIdSelector:
              matchControllerRef: true
            tags:
              kubernetes.io/role/elb: "1"
      patches:
        - type: PatchSet
          patchSetName: networkconfig
          # define eu-central-1b as zone & availabilityZone
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: metadata.labels.zone
          transforms:
            - type: string
              string:
                fmt: "%sb"
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: spec.forProvider.availabilityZone
          transforms:
            - type: string
              string:
                fmt: "%sb"
          # provide the subnetId for later use as status.subnetIds entry
        - type: ToCompositeFieldPath
          fromFieldPath: metadata.annotations[crossplane.io/external-name]
          toFieldPath: status.subnetIds[1]

    - name: subnet-public-eu-central-1c
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: Subnet
        metadata:
          labels:
            access: public
        spec:
          forProvider:
            mapPublicIpOnLaunch: true
            cidrBlock: 10.0.2.0/24
            vpcIdSelector:
              matchControllerRef: true
            tags:
              kubernetes.io/role/elb: "1"
      patches:
        - type: PatchSet
          patchSetName: networkconfig
          # define eu-central-1c as zone & availabilityZone
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: metadata.labels.zone
          transforms:
            - type: string
              string:
                fmt: "%sc"
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: spec.forProvider.availabilityZone
          transforms:
            - type: string
              string:
                fmt: "%sc"
          # provide the subnetId for later use as status.subnetIds entry
        - type: ToCompositeFieldPath
          fromFieldPath: metadata.annotations[crossplane.io/external-name]
          toFieldPath: status.subnetIds[2]  

    ### SecurityGroup & SecurityGroupRules Cluster API server
    - name: securitygroup-cluster
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: SecurityGroup
        metadata:
          labels:
            net.aws.crossplane.jonashackt.io: securitygroup-cluster
        spec:
          forProvider:
            description: cluster API server access
            name: securitygroup-cluster
            vpcIdSelector:
              matchControllerRef: true
      patches:
        - type: PatchSet
          patchSetName: networkconfig
        - fromFieldPath: spec.id
          toFieldPath: metadata.name
          # provide the securityGroupId for later use as status.securityGroupClusterIds entry
        - type: ToCompositeFieldPath
          fromFieldPath: metadata.annotations[crossplane.io/external-name]
          toFieldPath: status.securityGroupClusterIds[0]

    - name: securitygrouprule-cluster-inbound
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: SecurityGroupRule
        spec:
          forProvider:
            #description: Allow pods to communicate with the cluster API server & access API server from kubectl clients
            type: ingress
            cidrBlocks:
              - 0.0.0.0/0
            fromPort: 443
            toPort: 443
            protocol: tcp
            securityGroupIdSelector:
              matchLabels:
                net.aws.crossplane.jonashackt.io: securitygroup-cluster
      patches:
        - type: PatchSet
          patchSetName: networkconfig

    - name: securitygrouprule-cluster-outbound
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: SecurityGroupRule
        spec:
          forProvider:
            description: Allow internet access from the cluster API server
            type: egress
            cidrBlocks: # Destination
              - 0.0.0.0/0
            fromPort: 0
            toPort: 0
            protocol: tcp
            securityGroupIdSelector:
              matchLabels:
                net.aws.crossplane.jonashackt.io: securitygroup-cluster
      patches:
        - type: PatchSet
          patchSetName: networkconfig

    ### Route, RouteTable & RouteTableAssociations
    - name: route
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: Route
        spec:
          forProvider:
            destinationCidrBlock: 0.0.0.0/0
            gatewayIdSelector:
              matchControllerRef: true
            routeTableIdSelector:
              matchControllerRef: true
      patches:
        - type: PatchSet
          patchSetName: networkconfig

    - name: routeTable
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: RouteTable
        spec:
          forProvider:
            vpcIdSelector:
              matchControllerRef: true
      patches:
      - type: PatchSet
        patchSetName: networkconfig

    - name: mainRouteTableAssociation
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: MainRouteTableAssociation
        spec:
          forProvider:
            routeTableIdSelector:
              matchControllerRef: true
            vpcIdSelector:
              matchControllerRef: true
      patches:
        - type: PatchSet
          patchSetName: networkconfig

    - name: RouteTableAssociation-public-a
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: RouteTableAssociation
        spec:
          forProvider:
            routeTableIdSelector:
              matchControllerRef: true
            subnetIdSelector:
              matchControllerRef: true
              matchLabels:
                access: public
      patches:
        - type: PatchSet
          patchSetName: networkconfig
        # define eu-central-1a as subnetIdSelector.matchLabels.zone
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: spec.forProvider.subnetIdSelector.matchLabels.zone
          transforms:
            - type: string
              string:
                fmt: "%sa"

    - name: RouteTableAssociation-public-b
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: RouteTableAssociation
        spec:
          forProvider:
            routeTableIdSelector:
              matchControllerRef: true
            subnetIdSelector:
              matchControllerRef: true
              matchLabels:
                access: public
      patches:
        - type: PatchSet
          patchSetName: networkconfig
        # define eu-central-1b as subnetIdSelector.matchLabels.zone
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: spec.forProvider.subnetIdSelector.matchLabels.zone
          transforms:
            - type: string
              string:
                fmt: "%sb"

    - name: RouteTableAssociation-public-c
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: RouteTableAssociation
        spec:
          forProvider:
            routeTableIdSelector:
              matchControllerRef: true
            subnetIdSelector:
              matchControllerRef: true
              matchLabels:
                access: public
      patches:
        - type: PatchSet
          patchSetName: networkconfig
        # define eu-central-1c as subnetIdSelector.matchLabels.zone
        - type: FromCompositeFieldPath
          fromFieldPath: spec.parameters.region
          toFieldPath: spec.forProvider.subnetIdSelector.matchLabels.zone
          transforms:
            - type: string
              string:
                fmt: "%sc"
  ```
</details>


For the start, let's simply apply our first XRD, Composition and Claim manually like that:

```shell
# Networking XRD & Composition
kubectl apply -f apis/networking/definition.yaml
kubectl apply -f apis/networking/composition.yaml
# Precheck if Network works
kubectl apply -f examples/networking/claim.yaml
```

I found that the simplest way to follow what Crossplane is doing, is to look into the events ( via typing `:events`) in k9s:

![](docs/follow-crossplane-events-in-k9s.png)

And simply press `ENTER` to see the actual event message. This helped me a lot in the development process (no need to run `kubectl get crossplane` all the time and manually copy the CRD names to a `kubectl describe xyz-crd`).



Managed Resources need to reference other Managed Resources. For example, a `SecurityGroupRule` needs to reference a `SecurityGroup`:

```yaml
...
    ### SecurityGroups & Rules
    - name: securitygroup-nodepool
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: SecurityGroup
        spec:
          forProvider:
            description: Cluster communication with worker nodes
            name: securitygroup-nodepool
            vpcIdSelector:
              matchControllerRef: true

      patches:
        - type: PatchSet
          patchSetName: networkconfig
        - fromFieldPath: spec.id
          toFieldPath: metadata.name
          # provide the securityGroupId for later use as status.securityGroupIds entry
        - type: ToCompositeFieldPath
          fromFieldPath: metadata.annotations[crossplane.io/external-name]
          toFieldPath: status.securityGroupIds[0]

    - name: securitygroup-nodepool-rule
      base:
        apiVersion: ec2.aws.upbound.io/v1beta1
        kind: SecurityGroupRule
        spec:
          forProvider:
            type: egress
            cidrBlocks:
              - 0.0.0.0/0
            fromPort: 0
            protocol: tcp
            securityGroupIdSelector:
              matchLabels:
                net.aws.crossplane.jonashackt.io: securitygroup
            toPort: 0
      patches:
        - type: PatchSet
          patchSetName: networkconfig
          ...
```

In this example, we get the following error in our k8s events:

```shell
cannot resolve references: mg.Spec.ForProvider.SecurityGroupID: no resources matched selector
```

https://docs.crossplane.io/latest/concepts/managed-resources/#referencing-other-resources states

> Some fields in a managed resource may depend on values from other managed resources. For example a VM may need the name of a virtual network to use.

> Managed resources can reference other managed resources by external name, name reference or selector.

The problem is, we don't specify the `net.aws.crossplane.jonashackt.io: securitygroup` label on our `SecurityGroup`! Doing that the problem is gone:

```yaml
        kind: SecurityGroup
        metadata:
          labels:
            net.aws.crossplane.jonashackt.io: securitygroup
```


There should now be an event showing up containing `Successfully composed resources` in our `eks-vpc-j8s5k` XR.



### The EKS Cluster Composition

Can be found in `apis/eks/` directory:

* XRD: [`apis/eks/definition.yaml`](apis/eks/definition.yaml)

<details>
  <summary>expand full yaml</summary>

  ```yaml
  apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  # XRDs must be named 'x<plural>.<group>'
  name: xeksclusters.eks.aws.crossplane.jonashackt.io
spec:
  # This XRD defines an XR in the 'crossplane.jonashackt.io' API group.
  # The XR or Claim must use this group together with the spec.versions[0].name as it's apiVersion, like this:
  # 'crossplane.jonashackt.io/v1alpha1'
  group: eks.aws.crossplane.jonashackt.io
  
  # XR names should always be prefixed with an 'X'
  names:
    kind: XEKSCluster
    plural: xeksclusters
  # This type of XR offers a claim, which should have the same name without the 'X' prefix
  claimNames:
    kind: EKSCluster
    plural: ekscluster
  
  # default Composition when none is specified (must match metadata.name of a provided Composition)
  # e.g. in composition.yaml
  defaultCompositionRef:
    name: aws-eks

  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    # OpenAPI schema (like the one used by Kubernetes CRDs). Determines what fields
    # the XR (and claim) will have. Will be automatically extended by crossplane.
    # See https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/
    # for full CRD documentation and guide on how to write OpenAPI schemas
    schema:
      openAPIV3Schema:
        type: object
        properties:
          # defining input parameters
          spec:
            type: object
            properties:
              id:
                type: string
                description: ID of this Cluster that other objects will use to refer to it.
              parameters:
                type: object
                description: EKS configuration parameters.
                properties:
                  # Using subnetIds & securityGroupClusterIds from XNetworking to configure VPC
                  subnetIds:
                    type: array
                    items:
                      type: string
                  securityGroupClusterIds:
                    type: array
                    items:
                      type: string
                  region:
                    type: string
                  nodes:
                    type: object
                    description: EKS node configuration parameters.
                    properties:
                      count:
                        type: integer
                        description: Desired node count, from 1 to 10.
                    required:
                    - count
                required:
                - subnetIds
                - securityGroupClusterIds
                - region
                - nodes
            required:
            - id
            - parameters
          # defining return values
          status:
            type: object
            properties:
              clusterStatus:
                description: The status of the control plane
                type: string
              nodePoolStatus:
                description: The status of the node pool
                type: string
  ```
</details>

* Composition: [`apis/eks/composition.yaml`](apis/eks/composition.yaml)

<details>
  <summary>expand full yaml</summary>

  ```yaml
  apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: aws-eks
  labels:
    provider: aws
spec:
  compositeTypeRef:
    apiVersion: eks.aws.crossplane.jonashackt.io/v1alpha1
    kind: XEKSCluster
  
  writeConnectionSecretsToNamespace: crossplane-system

  patchSets:
  - name: clusterconfig
    patches:
    - fromFieldPath: spec.parameters.region
      toFieldPath: spec.forProvider.region

  resources:
    ### Cluster Configuration
    - name: eksCluster
      base:
        apiVersion: eks.aws.upbound.io/v1beta1
        kind: Cluster
        metadata:
          annotations:
            meta.upbound.io/example-id: eks/v1beta1/cluster
            uptest.upbound.io/timeout: "2400"
        spec:
          forProvider:
            roleArnSelector:
              matchControllerRef: true
              matchLabels:
                role: clusterRole
            vpcConfig:
              - endpointPrivateAccess: true
                endpointPublicAccess: true
      patches:
        - type: PatchSet
          patchSetName: clusterconfig
        - fromFieldPath: spec.id
          toFieldPath: metadata.name
        # Using the XNetworking defined securityGroupClusterIds & subnetIds for the vpcConfig
        - fromFieldPath: spec.parameters.securityGroupClusterIds
          toFieldPath: spec.forProvider.vpcConfig[0].securityGroupIds
        - fromFieldPath: spec.parameters.subnetIds
          toFieldPath: spec.forProvider.vpcConfig[0].subnetIds

        - type: ToCompositeFieldPath
          fromFieldPath: status.atProvider.status
          toFieldPath: status.clusterStatus    
      readinessChecks:
        - type: MatchString
          fieldPath: status.atProvider.status
          matchString: ACTIVE

    - name: kubernetesClusterAuth
      base:
        apiVersion: eks.aws.upbound.io/v1beta1
        kind: ClusterAuth
        spec:
          forProvider:
            clusterNameSelector:
              matchControllerRef: true
      patches:
        - type: PatchSet
          patchSetName: clusterconfig
        - fromFieldPath: spec.writeConnectionSecretToRef.namespace
          toFieldPath: spec.writeConnectionSecretToRef.namespace
        - fromFieldPath: spec.id
          toFieldPath: spec.writeConnectionSecretToRef.name
          transforms:
            - type: string
              string:
                fmt: "%s-access"
      connectionDetails:
        - fromConnectionSecretKey: kubeconfig

    ### Cluster Role and Policies
    - name: clusterRole
      base:
        apiVersion: iam.aws.upbound.io/v1beta1
        kind: Role
        metadata:
          labels:
            role: clusterRole
        spec:
          forProvider:
            assumeRolePolicy: |
              {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": [
                                "eks.amazonaws.com"
                            ]
                        },
                        "Action": [
                            "sts:AssumeRole"
                        ]
                    }
                ]
              }
      
    
    - name: clusterRolePolicyAttachment
      base:
        apiVersion: iam.aws.upbound.io/v1beta1
        kind: RolePolicyAttachment
        spec:
          forProvider:
            policyArn: arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
            roleSelector:
              matchControllerRef: true
              matchLabels:
                role: clusterRole


    ### NodeGroup Configuration
    - name: nodeGroupPublic
      base:
        apiVersion: eks.aws.upbound.io/v1beta1
        kind: NodeGroup
        spec:
          forProvider:
            clusterNameSelector:
              matchControllerRef: true
            nodeRoleArnSelector:
              matchControllerRef: true
              matchLabels:
                role: nodegroup
            subnetIdSelector:
              matchLabels:
                access: public
            scalingConfig:
              - minSize: 1
                maxSize: 10
                desiredSize: 1
            instanceTypes: # TODO: we can support to have that parameterized also
              - t3.medium
      patches:
        - type: PatchSet
          patchSetName: clusterconfig
        - fromFieldPath: spec.parameters.nodes.count
          toFieldPath: spec.forProvider.scalingConfig[0].desiredSize
        - fromFieldPath: spec.id
          toFieldPath: spec.forProvider.subnetIdSelector.matchLabels[net.aws.crossplane.jonashackt.io/network-id]
        - type: ToCompositeFieldPath
          fromFieldPath: status.atProvider.status
          toFieldPath: status.nodePoolStatus  
      readinessChecks:
      - type: MatchString
        fieldPath: status.atProvider.status
        matchString: ACTIVE

    ### Node Role and Policies
    - name: nodegroupRole
      base:
        apiVersion: iam.aws.upbound.io/v1beta1
        kind: Role
        metadata:
          labels:
            role: nodegroup
        spec:
          forProvider:
            assumeRolePolicy: |
              {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": [
                                "ec2.amazonaws.com"
                            ]
                        },
                        "Action": [
                            "sts:AssumeRole"
                        ]
                    }
                ]
              }
      

    - name: workerNodeRolePolicyAttachment
      base:
        apiVersion: iam.aws.upbound.io/v1beta1
        kind: RolePolicyAttachment
        spec:
          forProvider:
            policyArn: arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
            roleSelector:
              matchControllerRef: true
              matchLabels:
                role: nodegroup
      

    - name: cniRolePolicyAttachment
      base:
        apiVersion: iam.aws.upbound.io/v1beta1
        kind: RolePolicyAttachment
        spec:
          forProvider:
            policyArn: arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
            roleSelector:
              matchControllerRef: true
              matchLabels:
                role: nodegroup
      
    - name: containerRegistryRolePolicyAttachment
      base:
        apiVersion: iam.aws.upbound.io/v1beta1
        kind: RolePolicyAttachment
        spec:
          forProvider:
            policyArn: arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
            roleSelector:
              matchControllerRef: true
              matchLabels:
                role: nodegroup
      
  ```
</details>

For testing we simply use `kubectl apply -f`:


```shell
# EKS XRD & Composition
kubectl apply -f apis/eks/definition.yaml
kubectl apply -f apis/eks/composition.yaml

# If you choose this example (non-nested) claim, be sure to change the subnetIds and securitygroupid according the the Networking claim executed before!

# Precheck if EKSCluster works
kubectl apply -f examples/eks/claim.yaml 
```

Errors in the events like this are normal, since the EKS Cluster needs it's time to be provisioned before NodeGroups etc. can be assigned:

```shell
cannot resolve references: mg.Spec.ForProvider.ClusterName: referenced field was empty (referenced resource may not yet be ready) 
```

This also shows up in the AWS console:

![](docs/eks-cluster-initial-provisioning.png)


Now if the `NodeGroup` comes up with the following

```shell
cannot resolve references: mg.Spec.ForProvider.SubnetIds: no resources matched selector 
```

there's a problem, where the NodeGroup can't find it's SubnetIds.

```yaml
    - name: nodeGroupPublic
      base:
        apiVersion: eks.aws.upbound.io/v1beta1
        kind: NodeGroup
        spec:
          forProvider:
            clusterNameSelector:
              matchControllerRef: true
            nodeRoleArnSelector:
              matchControllerRef: true
              matchLabels:
                role: nodegroup
            subnetIdSelector:
              matchLabels:
                access: public
            scalingConfig:
              - minSize: 1
                maxSize: 10
                desiredSize: 1
            instanceTypes: # TODO: we can support to have that parameterized also
              - t3.medium
      patches:
        - type: PatchSet
          patchSetName: clusterconfig
        - fromFieldPath: spec.parameters.nodes.count
          toFieldPath: spec.forProvider.scalingConfig[0].desiredSize
        - fromFieldPath: spec.id
          toFieldPath: spec.forProvider.subnetIdSelector.matchLabels[aws.crossplane.jonashackt.io/network-id]
          ...
```

That's because the label of all networking components changed to `net.aws.crossplane.jonashackt.io/network-id`. So let's fix that!

Now finally the NodeGroups are correctly assigned to the EKS cluster:

The `Successfully composed resources` message in the event `xekscluster/deploy-target-eks-cb87r` looks promising:

![](docs/eks-cluster-with-nodegroups.png)



### The nested XR for Networking & EKS Cluster Compositions

Can be found in `apis/` directory:

* XRD: [`apis/definition.yaml`](apis/definition.yaml)
* Composition: [`apis/composition.yaml`](apis/composition.yaml)

With this Composition we're able to use both pre-defined Compositions `XNetworking` and `XEKSCluster` and thus implement a nested Composite Resource:

```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: kubernetes-cluster
spec:
  compositeTypeRef:
    apiVersion: k8s.crossplane.jonashackt.io/v1alpha1
    kind: XKubernetesCluster
  
  writeConnectionSecretsToNamespace: crossplane-system

  resources:
    ### Nested use of XNetworking XR
    - name: compositeNetworkEKS
      base:
        apiVersion: net.aws.crossplane.jonashackt.io/v1alpha1
        kind: XNetworking
      patches:
        - fromFieldPath: spec.id
          toFieldPath: spec.id
        - fromFieldPath: spec.parameters.region
          toFieldPath: spec.parameters.region
        # provide the subnetIds & securityGroupIds for later use
        - type: ToCompositeFieldPath
          fromFieldPath: status.subnetIds
          toFieldPath: status.subnetIds
          policy:
            fromFieldPath: Required
        - type: ToCompositeFieldPath
          fromFieldPath: status.securityGroupIds
          toFieldPath: status.securityGroupIds
          policy:
            fromFieldPath: Required
    
    ### Nested use of XEKSCluster XR
    - name: compositeClusterEKS
      base:
        apiVersion: eks.aws.crossplane.jonashackt.io/v1alpha1
        kind: XEKSCluster
      connectionDetails:
        - fromConnectionSecretKey: kubeconfig
      patches:
        - fromFieldPath: spec.id
          toFieldPath: spec.id
        - fromFieldPath: spec.id
          toFieldPath: metadata.annotations[crossplane.io/external-name]
        - fromFieldPath: metadata.uid
          toFieldPath: spec.writeConnectionSecretToRef.name
          transforms:
            - type: string
              string:
                fmt: "%s-eks"
        - fromFieldPath: spec.writeConnectionSecretToRef.namespace
          toFieldPath: spec.writeConnectionSecretToRef.namespace
        - fromFieldPath: spec.parameters.region
          toFieldPath: spec.parameters.region
        - fromFieldPath: spec.parameters.nodes.count
          toFieldPath: spec.parameters.nodes.count
        - fromFieldPath: status.subnetIds
          toFieldPath: spec.parameters.subnetIds
          policy:
            fromFieldPath: Required
        - fromFieldPath: status.securityGroupIds
          toFieldPath: spec.parameters.securityGroupIds
          policy:
            fromFieldPath: Required
```

For the start, let's simply apply our first XRD, Composition and Claim manually like that:

```shell
# Nested XRD & Composition
kubectl apply -f apis/definition.yaml
kubectl apply -f apis/composition.yaml

# Check if full Cluster provisioning works
kubectl apply -f examples/claim.yaml
```



### Accessing the Crossplane provisioned EKS cluster

https://docs.crossplane.io/knowledge-base/guides/connection-details/

In our eks cluster [claim](upbound/provider-aws/apis/eks/claim.yaml) we defined a 

```yaml
  writeConnectionSecretToRef:
    name: eks-cluster-kubeconfig
```

inside our nested claim. This will create a k8s `Secret` called `eks-cluster-kubeconfig`, where the kubeconfig will be stored.

Let's extract the kubeconfig:

```shell
kubectl get secret eks-cluster-kubeconfig -o jsonpath='{.data.kubeconfig}' | base64 --decode > ekskubeconfig
```

Now integrate the contents of the `ekskubeconfig` file into your `~/.kube/config` (better with VSCode!) and switch over to the new kube context e.g. using https://github.com/ahmetb/kubectx. If you're on the new context of our Crossplane bootstrapped EKS cluster, check if everything works:

```shell
$ kubectl get nodes
NAME                                          STATUS   ROLES    AGE   VERSION
ip-10-0-0-173.eu-central-1.compute.internal   Ready    <none>   34m   v1.29.0-eks-5e0fdde
ip-10-0-1-149.eu-central-1.compute.internal   Ready    <none>   34m   v1.29.0-eks-5e0fdde
ip-10-0-2-90.eu-central-1.compute.internal    Ready    <none>   34m   v1.29.0-eks-5e0fdde
```



# Testing the Managed Resources Rendering with kuttl

As described in this project: https://github.com/jonashackt/crossplane-kuttl we'll use https://kuttl.dev to create tests for our EKS Cluster Composition.

```shell
# Run kuttl tests
kubectl kuttl test
```

To run multiple tests, you don't need to setup kind and Crossplane incl. it's Providers every time simply run:

```shell
# Only once:
kubectl kuttl test --skip-cluster-delete
# and the following runs:
kubectl kuttl test --start-kind=false
```

Tests can be found in the exact reflective order as in `apis` under `tests/compositions`.

If an error occurs like `key is missing from map`:

```shell
case.go:366: resource VPC:/: .spec.forProvider.instanceTenancy: key is missing from map
```

one needs to delete that entry from the `01-assert.yaml`.

Even if something appears like 

```shell
resource Subnet:/: .metadata.labels.zone: value mismatch, expected: eu-central-1a != actual: eu-central-1b
```

Fix the `key is missing from map` first! Then the others might disappear.


Also for better readability, we run the kuttl tests one after another by using the `parallel: 1` configuration in the [`kuttl-test.yaml](kuttl-test.yaml):

```yaml
...
parallel: 1 # use parallel: 1 to execute one test after another (e.g. for better readability in CI logs)
```



# Building a Configuration Package as OCI container

https://docs.crossplane.io/latest/concepts/packages/#create-a-configuration

https://morningspace.medium.com/build-publish-and-install-crossplane-package-5e4b74a3ee37


### Install Crossplane CLI

https://github.com/crossplane/crossplane/releases/tag/v1.15.0

> Enhancements to the Crossplane CLI: New subcommands like `crossplane beta validate` for schema validation, `crossplane beta top` for resource utilization views similar to `kubectl top pods`, and `crossplane beta convert` for converting resources to newer formats or configurations.

Install crossplane CLI:

```shell
curl -sL "https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh" |sh
```

If that produces an error like `Failed to download Crossplane CLI. Please make sure version current exists on channel stable.`, try to manually craft the download link:

```shell
curl --output crank "https://releases.crossplane.io/stable/current/bin/linux_amd64/crank"
chmod +x crank
sudo mv crank /usr/local/bin/crossplane
```

Be sure to have the `v1.15.0` version installed as a minimum, otherwise the `crossplane beta validate` command won't work:

```shell
crossplane --version
v1.15.0
```

### The crossplane.yaml

As [stated in the docs](https://docs.crossplane.io/latest/concepts/packages/#create-a-configuration) Crossplane has a feature where one can create Configuration Packages containing specific Compositions packaged in a OCI container. So let's build a Configuration Package from this EKS setup here.

First we need a [`crossplane.yaml`](crossplane.yaml):

```yaml
apiVersion: meta.pkg.crossplane.io/v1alpha1
kind: Configuration
metadata:
  name: crossplane-eks-cluster
  annotations:
    # Set the annotations defining the maintainer, source, license, and description of your Configuration
    meta.crossplane.io/maintainer: Jonas Hecht iam@jonashackt.io
    meta.crossplane.io/source: github.com/jonashackt/crossplane-eks-cluster
    # Set the license of your Configuration
    meta.crossplane.io/license: MIT
    meta.crossplane.io/description: |
      Crossplane Configuration delivering CRDs to provision AWS EKS clusters.
    meta.crossplane.io/readme: |
      It's a Nested Composition with separate AWS Networking & EKS cluster setups.
spec:
  dependsOn:
    - provider: xpkg.upbound.io/upbound/provider-aws-ec2
      version: ">=v1.1.1"
    - provider: xpkg.upbound.io/upbound/provider-aws-iam
      version: ">=v1.1.1"
    - provider: xpkg.upbound.io/upbound/provider-aws-eks
      version: ">=v1.1.1"
  crossplane:
    version: ">=v1.15.1-0"
```

Don't forget to add the `metadate.annotations` in order to prevent the error `crossplane: error: failed to build package: not exactly one package meta type` - see also https://stackoverflow.com/questions/78200917/crossplane-error-failed-to-build-package-not-exactly-one-package-meta-type

Also we should define on which providers our Configuration depends on - and also on which Crossplane version.


There's also a template one could use to create the crossplane.yaml using the crossplane CLI: https://docs.crossplane.io/latest/cli/command-reference/#beta-xpkg-init 

```shell
crossplane beta xpkg init crossplane-eks-cluster configuration-template
```

The command uses the following template: https://github.com/crossplane/configuration-template (one could provide arbitrary repositories to the command).


### Building the Package using Crossplane CLI

The Crossplane CLI [has the right command for us](https://docs.crossplane.io/latest/concepts/packages/#build-the-package):

```shell
crossplane xpkg build --package-root=. --examples-root="./examples" --ignore=".github/workflows/*,crossplane/install/*,crossplane/provider/*,kuttl-test.yaml,tests/compositions/eks/*,tests/compositions/networking/*" --verbose
```

Note that including YAML files that aren’t Compositions or CompositeResourceDefinitions, including Claims isn’t supported.

This can be done by appending `--ignore="file.xyz,directory/*"`, which will ignore a file `file.xyz` and all files in directory `directory`. [Sadly, ingoring directories completely isn't supported right now](https://docs.crossplane.io/latest/concepts/packages/#build-the-package) - so we need to define all our kuttl test directories respectively.

Also appending `--verbose` makes a lot of sense to see what's going on.


### Pushing the Package file .xpkg to GitHub Container Registry

There's also a `crossplane xpkg push` command to publish the Configuration package. So let's create a new GitHub package matching our repository:

```shell
crossplane xpkg push ghcr.io/jonashackt/crossplane-eks-cluster:v0.0.1
```

You can leverage the Container image tag as version number for your Configuration here.

If the command gives the following error, we need to setup Authentication for your Docker Registry:

```shell
crossplane: error: failed to push package file crossplane-eks-cluster-7badc365c06a.xpkg: Post "https://ghcr.io/v2/jonashackt/crossplane-eks-cluster/blobs/uploads/": GET https://ghcr.io/token?scope=repository%3Ajonashackt%2Fcrossplane-eks-cluster%3Apull&scope=repository%3Ajonashackt%2Fcrossplane-eks-cluster%3Apush%2Cpull&service=ghcr.io: DENIED: requested access to the resource is denied
```

The ` crossplane xpkg push --help` helps us:

> Credentials for the registry are automatically retrieved from xpkg
login and dockers configuration as fallback.

So we need to login to GitHub Container Registry first in order to be able to push our OCI image:

```shell
echo $CR_PAT | docker login ghcr.io -u YourAccountOrGHOrgaNameHere --password-stdin
```

Make sure to use a Personal Access Token as described in this post https://www.codecentric.de/wissens-hub/blog/github-container-registry with the following scopes (`repo`, `write:packages` and `delete:packages`):

![](docs/github-container-registry-pat-scopes.png)

Additionally we need to add the domain configuration like this: `--domain=https://ghcr.io`. Otherwise the default domain is `upbound.io` which will lead to non pushed Configurations - only visible via the `verbose` flag.

With this our `crossplane xpkg push` command should work as expected:

```shell
$ crossplane xpkg push ghcr.io/jonashackt/crossplane-eks-cluster:v0.0.1 --domain=https://ghcr.io --verbose

2024-03-21T16:39:48+01:00	DEBUG	Found package in directory	{"path": "crossplane-eks-cluster-7badc365c06a.xpkg"}
2024-03-21T16:39:48+01:00	DEBUG	Getting credentials for server	{"serverURL": "ghcr.io"}
2024-03-21T16:39:48+01:00	DEBUG	No profile specified, using default profile
2024-03-21T16:39:49+01:00	DEBUG	Pushed package	{"path": "crossplane-eks-cluster-7badc365c06a.xpkg", "ref": "ghcr.io/jonashackt/crossplane-eks-cluster:v0.0.1"}
```

Now head over to your GitHub Organisation's `Packages` tab and search for the newly created package:

![](docs/github-container-registry-package-connect-repository.png)

Click onto the package and connect the GitHub Repository.

Also - on the right - click on `Package settings` and scroll down to the `Danger Zone`. There click on `Change visibility` and change it to public. Now your Crossplane Configuration should be available for download without login.

If everything went fine, the package / OCI image should now be visible at your repository:

![](docs/github-container-registry-package-visible.png)


### Build & Publish Crossplane Configuration Packages automatically with GitHub Actions

https://docs.upbound.io/xp-arch-framework/building-apis/building-apis-configurations/#set-up-a-build-pipeline-with-github

So let's finally do it all automatically on Composition code changes (git commit/push) using GitHub Actions. We simply extend our workflow at [.github/workflows/test-composition-and-publish-to-ghcr.yml](.github/workflows/test-composition-and-publish-to-ghcr.yml) and do all the steps from above:

```yaml
name: publish

on: [push]

env:
  GHCR_PAT: ${{ secrets.GHCR_PAT }}
  CONFIGURATION_VERSION: "v0.0.2"

jobs:
  resouces-rendering-test:
    ...

jobs:
  build-configuration-and-publish-to-ghcr:
    needs: resouces-rendering-test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GHCR_PAT }}

      - name: Install Crossplane CLI
        run: |
          curl -sL "https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh" |sh
          sudo mv crossplane /usr/local/bin

      - name: Build Crossplane Configuration package & publish it to GitHub Container Registry
        run: |
          echo "### Build Configuration .xpkg file"
          crossplane xpkg build --package-root=. --examples-root="./examples" --ignore=".github/workflows/*,crossplane/install/*,crossplane/provider/*,kuttl-test.yaml,tests/compositions/eks/*,tests/compositions/networking/*" --verbose

          echo "### Publish as OCI image to GHCR"
          crossplane xpkg push "ghcr.io/jonashackt/crossplane-eks-cluster:$CONFIGURATION_VERSION" --domain=https://ghcr.io --verbose
```

As we added the `.github/workflows` directory with a workflow yaml file, the `crossplane xpkg build` command also tries to include it. Therefore the command locally need to exclude the workflow file also:

```shell
crossplane xpkg build --package-root=. --examples-root="./examples" --ignore=".github/workflows/*,crossplane/install/*,crossplane/provider/*,kuttl-test.yaml,tests/compositions/eks/*,tests/compositions/networking/*" --verbose
```

`--ignore=".github/*` won't work, since the command doesn't support to exclude directories - only wildcards IN directories.

Also to prevent the following error:

```shell
crossplane: error: failed to push package file crossplane-eks-cluster-7badc365c06a.xpkg: PUT https://ghcr.io/v2/jonashackt/crossplane-eks-cluster/manifests/v0.0.2: DENIED: installation not allowed to Write organization package
```

we use the Personal Access Token (PAT) we already created above also in our GitHub Actions Workflow instead of the default `GITHUB_TOKEN` in order to have the correct permissions. Therefore create it as a new Repository Secret:

![](docs/create-repository-secret.png)

With this we should also be able to use a ENV var for our Configuration version or even `latest`.

To prevent the `build-configuration-and-publish-to-ghcr` step from running before the test job successfully finished, [we use the `needs: resouces-rendering-test` keyword as described here](https://stackoverflow.com/a/65698892/4964553).

Now our Pipeline should look like this:

![](docs/only-publish-when-test-successful-pipeline.png)