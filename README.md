
# (Google) Cloud on a  Budget using Terraform
* Have you ever wanted to understand how applications ( Websites ) are deployed in the cloud?
* Know how infrastructure components ( Servers, Storage, Networks ) are created in the cloud?
* Wondered how secrets and data is handled securely in the cloud? 

If the answer to any of these questions is yes, keep reading.
If the answer is no, do you like free stuff? Maybe not, but you may learn something new if you keep reading.

Before proceeding any further let us define cloud simply as
> Hardware and software services from a provider on the internet.
This means you can have computing resources and hardware made available to you on-demand.
You are no longer restricted to resources on your personal computer, company's network or any other computers that you directly manage.

There are a number of cloud providers each with differing popularity, features and price points.  
The decision on which provider to use is daunting task for both individuals and companies to make given the sheer number of offerings available in the market.
However, fret not dear reader. In the following sections I will guide you through elements to look out for when choosing a provider. I will also take you through my thought process.  

After quite a bit of (unscientific) research and testing I settled on Google Cloud Platform as my _major cloud provider_ of choice.
Here's why:
1. It is ~~cheap~~ value for money and cheaper than comparable providers
2. It has a very generous free tier plus the 300$ credit for new signups is not bad either.
3. It has a massive feature set
4. It has clean and intuitive interface that is easy to navigate
5. It provides your with a free powerful computer accessible from your browser for 50hrs every week. They call this [*Cloud Shell*](https://cloud.google.com/shell)

Now for the flip side. Cloud can get very expensive.   
[Seriously](https://dev-blog.tomilkieway.com/72k-1/).  
A cloud bill can get high enough in just a single day to effectively [put you out of business](https://dev-blog.tomilkieway.com/72k-1/).
Careful planning, tight control of resources, budget alerts & limits are necessary to manage your spending.  
This is why it is more common to find _major cloud providers_ mostly being used by enterprises.  
Hobbyists, students, startups and individuals have to make do with smaller _budget cloud providers_ whose product offering is more limited but much cheaper and  sufficient for testing out and running applications.  
A product or application in development is typified by lots of changes, very low traffic and most likely does not generate any revenue. An ideal situation should allow us to develop this product for free or keep the costs as low as possible. Same goes for educational projects.  
Given that this is the case, let us set a target of **10$** a month spend as the upper limit for the amount we are willing/able to spend on our hobby projects.  
This is possible with a few _budget cloud providers_ but let us challenge ourselves to do this on Google Cloud.
We'll definitely have to cut some corners but we'll take note of when we do this. We will also try as much as possible to follow best practices recommend by Google itself.

## The Challenge

Create Infrastructure on Google Cloud for less than 10$ a month that meets the following criteria;
1. Creates a Cluster of (at least 3) Machines
2. Automated Process
3. Secure, Private and follows best practices endorsed by Google
The example below can be run by anyone, just click the button and you're good to go.  

However, understanding the terminology used in subsequent section requires some basic knowledge and/or prior experience of deploying an application.  
You may safely skim if all this is new to you.

[![Open this project in Cloud Shell](http://gstatic.com/cloudssh/images/open-btn.png)](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/nufailtd/terraform-budget-gcp&open_in_editor=seed_project/terraform.tfvars)

Of course, the only correct answers on *how* we can meet the above criteria is by using;
1. [Kubernetes](https://kubernetes.io/)
2. [Terraform](https://www.terraform.io/)
3. [Vault](https://www.vaultproject.io/)

Well...not really, but, these are some of the tools you can use to achieve the above.

##  The How
In order to achieve our goal, we need to first identify the Cloud Provider offerings we would like, establish the base cost then look for ways to cut costs as much as we possibly can.
Given that the Google Cloud Platform offers a generous free tier of products, we shall push our utility of these products to the maximum.

We make use of the following excellent open source projects to aid us in our task. Give them a look sometime.
* [Terraform](https://www.terraform.io/) - Declaratively provision and manage or infrastructure.
* [Vault](https://www.vaultproject.io/) - Storing and Managing access to our secrets.
* [Traefik](https://traefik.io/) - Routing traffic to our cluster.
* [Pomerium](https://www.pomerium.io/) - Secures access to our applications.
* [Chisel](https://github.com/jpillora/chisel) - Secure tunnelled connections into our cluster.
* [ExternalDNS](https://github.com/kubernetes-sigs/external-dns) - Mapping our domain to our cluster.
* [CertManager](https://cert-manager.io/) - Automatically creating certificates for our domains.
* [Berglas](https://github.com/GoogleCloudPlatform/berglas) - Store and retrieve secrets in Google Cloud.

## The What

| Product | Purpose | Type | Cost $ | Our Choice | Our Cost $ | Notes |
| :-- | :-- | :--|:-- |:-- |:-- |:-- |
| [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine) (GKE) | Manage our cluster of machines (nodes) and applications | Regional Cluster | 72.00 | Zonal Cluster |  0.00 |  A regional cluster is encouraged to mitigate against outages in a single zone but costs *72$*. The first zonal cluster is free. We will use a secure private cluster with no public ips and no internet access by default. |
| [Compute Engine](https://cloud.google.com/compute)  | These are the actual machines in our cluster. Our applications use resources in these machines. | e2-micro  | 6.11 | e2-micro pvm |  1.83 | We use the cheaper pre-emptible machines because GKE will take care of re-creating them when they are terminated |
| [Cloud Load Balancer](https://cloud.google.com/load-balancing) | Route requests from our users to our applications in the cluster. | External | 18.00 | Google Container f1-micro | 0.00 | GCP provides an always free f1-micro instance. We will use this instance together with traefik to route [traefik](https://traefik.io/) into our cluster. |
| [Cloud NAT](https://cloud.google.com/nat) | Access the internet from machines inside our cluster | Google Managed  | 1.008 | Custom NAT using f1-micro instance | 0.00| GCP provides an always free Public IP. We will use this IP attached to our free f1-micro as the next hop when connecting to the internet from our private cluster  |
| [Cloud KMS](https://cloud.google.com/security-key-management) | Encrypt secret data specifically the vault root token and unseal keys. | Cloud Managed | 0.06 | - | 0.06 | Cloud KMS is responsible for creating and managing keys for encrypting sensitive data. Storage of the encrypted data is left to you. |
| [Google Secrets Manager]() | Encryption, Storage, and access control of secret data.| Google Managed |0.06| - | 0.06 |Store sensitive data to be used by other infrastructure elements e.g. CloudRun |
| [Cloud Run](https://cloud.google.com/run) | Deploy [Vault](https://www.vaultproject.io/) and a couple of containerized applications including a [Ghost](https://github.com/TryGhost/Ghost) Blog. | 1CPU-256MB | - | 1CPU-256MB |0.00 | GCP has a generous free tier for cloudrun. Our usage will most likely fall within this tier. |
| [CloudSQL](https://cloud.google.com/sql) | MySQL Database for our Ghost Blog running in Cloudrun  | db-f1-micro | 10.17 | docker-mysql |0.00 | We will use a mysql docker container running in our free f1-micro instance. |
| [Serverless VPC Access Connector](https://cloud.google.com/vpc/docs/serverless-vpc-access) | Access infrastructure in our Virtual Private Cloud from our Cloudrun services. | Google Managed| 6.11| [Chisel](https://github.com/jpillora/chisel) Tunnel |0.00 | We use [Chisel](https://github.com/jpillora/chisel) to create a secure tunnel from the f1-micro instance to any containers that may need access to resources in our private network  |
| [Cloud IAP](https://cloud.google.com/iap) | Secures access to your applications using a single-sign-on flow.| Google Managed| 0.00 | Pomerium IAP |0.00 | We use [Pomerium](https://www.pomerium.io/) Identity Aware Proxy because it supports Google as well as external Identity Providers. |
| [Google Domains](https://domains.google/) | Provides the address for your website | Any domain charged per year | 9.00 | Freenom | 0.00 | We use [Freenom](https://www.freenom.com) to obtain a free .tk, .ml, .ga, .cf or .gk domain.|

## Monthly Cost Estimate

| Service Description | SKU Description | Usage amount | Usage unit | Cost $ |
| :-- | :-- | :--|:-- |:-- |
| Compute Engine | Preemptible E2 Instance Core| 727.428 |hour |4.76 |
| Compute Engine | Preemptible E2 Instance Ram | 2909.814 |gibibyte-hour |2.55 |
| Compute Engine | Storage PD Capacity| 70 |gibibyte month |1.60 |
| Kubernetes Engine | Zonal Kubernetes Clusters | 727.428 |hour |0.00 |
| Cloud Key Management Service | Active software symmetric key versions | 3 |active key versions |0.18|
| Cloud Key Management Service | Cryptographic operations with a software symmetric key | 150000 |active key versions |0.45|
| Secret Manager | Secret version replica storage | 6 |month |0.36 |
| Secret Manager| Secret access operations| 300 |count | 0.00 |
| | **Total** |  | month | **9.90** |

The estimate above assumes the following;
* 1 Zonal Kubernetes Cluster
* 4 e2-micro instances
* 1 f1-micro instance
* 3 KMS keys
* 1 Static IP
* 1 Vault CloudRun Service
* 1 Ghost Blog Cloud Run service
* 6 Secret Versions
It also assumes that you will not be transferring massive amounts of data outside your cluster i.e. a test cluster so that network costs will stay well within the free tier.
You may tweak these values to your liking but be wary of the cost implications.
## Summary
The recommended way to set up the infrastructure is by using the listed GCP product offerings for achieving the desired goals above.
You should almost always observe this when deploying production workloads.
Our choices have been deliberately made to cut down costs and sacrifice a number of features and some best practices to achieve our goal.
However our efforts to extract maximum value from our resources have also led to some new and  interesting discoveries.
Head on  over to this [repo](../../../terraform-budget-gcp) to create the infrastructure and share your experience of deploying on Google Cloud Platform on a budget, which issues were encountered and what improvements can be made.
Head over to this repo to create the 
