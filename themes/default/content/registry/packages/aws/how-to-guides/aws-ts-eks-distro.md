---
title: "Amazon EKS Distro Cluster | TypeScript"
h1: "Amazon EKS Distro Cluster"
linktitle: "Amazon EKS Distro Cluster"
meta_desc: "Amazon EKS Distro Cluster How-to Guide using TypeScript"
no_edit_this_page: true
cloud: aws
language: ts
layout: package
---

<!-- WARNING: this page was generated by a tool. Do not edit it by hand. -->
<!-- To change it, please see https://github.com/pulumi/docs/tree/master/tools/mktutorial. -->

<p class="mb-4 flex">
    <a class="flex flex-wrap items-center rounded-md font-display text-lg text-white bg-blue-600 border-2 border-blue-600 px-2 mr-2 whitespace-no-wrap hover:text-white" style="height: 45px;" href="https://github.com/pulumi/examples/tree/master/aws-ts-eks-distro" target="_blank">
        <span><i class="fab fa-github pr-2"></i> View Code</span>
    </a>
    <a href="https://app.pulumi.com/new?template=https://github.com/pulumi/examples/blob/master/aws-ts-eks-distro/README.md" target="_blank">
        <img src="https://get.pulumi.com/new/button.svg" alt="Deploy">
    </a>
</p>


This example deploys an Amazon EKS Distro cluster using a [dynamic provider](https://www.pulumi.com/docs/intro/concepts/resources/#dynamicproviders) which utilizes [kops](https://github.com/kubernetes/kops)

## Deploying the App

To deploy your infrastructure, follow the below steps.

### Prerequisites

1. [Install Pulumi](https://www.pulumi.com/docs/get-started/install/)
2. [Install Node.js](https://nodejs.org/en/download/)
3. [Install Kops](https://kops.sigs.k8s.io/getting_started/install/)
4. [Configure AWS Credentials](https://www.pulumi.com/docs/intro/cloud-providers/aws/setup/)
5. [Install `aws-iam-authenticator`](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)

### Steps

After cloning this repo, from this working directory, run these commands:

1. Install the required Node.js packages:

    ```bash
    $ npm install
    ```

2. Create a new stack, which is an isolated deployment target for this example:

    ```bash
    $ pulumi stack init
    ```

3. Set the required configuration variables for this program:

    ```bash
    $ pulumi config set aws:region us-west-2
    ```

4. Stand up the EKS cluster, which will also deploy the Kubernetes Dashboard:

    ```bash
    $ pulumi up
    ```

5. After 10-15 minutes, your cluster will be ready, and the kubeconfig JSON you'll use to connect to the cluster will
   be available as an output. You can save this kubeconfig to a file like so:

    ```bash
    $ pulumi stack output kubeconfig --show-secrets >kubeconfig.json
    ```

    Once you have this file in hand, you can interact with your new cluster as usual via `kubectl`:

    ```bash
    $ KUBECONFIG=./kubeconfig.json kubectl get nodes
    ```


7. From there, feel free to experiment. Make edits and run `pulumi up` to incrementally update your stack.
   For example, in order to deploy a Helm chart into your cluster, import the `@pulumi/kubernetes/helm` package,
   add a `Chart` resource that targets the EKS cluster to `index.ts`, and run `pulumi up`. Note that the Helm client
   must be set up in order for the chart to deploy. For more details, see the [Prerequisites](https://github.com/pulumi/examples/blob/master/aws-ts-eks-distro/#prerequisites) list.

    ```typescript
    import * as helm from "@pulumi/kubernetes/helm";

    // ... existing code here ...

    const myk8s = new k8s.Provider("myk8s", {
        kubeconfig: cluster.kubeconfig.apply(JSON.stringify),
    });

    const postgres = new helm.v2.Chart("postgres", {
        // stable/postgresql@0.15.0
        repo: "stable",
        chart: "postgresql",
        version: "0.15.0",
        values: {
            // Use a stable password.
            postgresPassword: "some-password",
            // Expose the postgres server via a load balancer.
            service: {
                type: "LoadBalancer",
            },
        },
    }, { providers: { kubernetes: myk8s } });
    ```

    Once the chart has been deployed, you can find its public, load-balanced endpoint via the Kubernetes Dashboard.

8. Once you've finished experimenting, tear down your stack's resources by destroying and removing it:

    ```bash
    $ pulumi destroy --yes
    $ pulumi stack rm --yes
    ```
