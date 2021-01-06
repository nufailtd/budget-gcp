
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
5. It provides your with a free powerful computer in their cloud for 50hrs every week. They call this *Cloud Shell*

Now for the flip side. Cloud can get very expensive.   
Seriously.  
A cloud bill can get high enough in just a single day to effectively put you out of business.
Careful planning, tight control of resources, budget alerts & limits are necessary to manage your spending.  
This is why it is more common to find _major cloud providers_ mostly being used by enterprises.  
Hobbyists, students, startups and individuals have to make do with smaller _budget cloud providers_ whose product offering is more limited but much cheaper and  sufficient for testing out and running applications.  
A product or application in development is typified by lots of changes, very low traffic and most likely does not generate any revenue. An ideal situation should allow us to develop this product for free or keep the costs as low as possible.  
Same goes for educational projects. 
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

Of course, the only correct answers on *how* we can meet the above criteria is by using;
1. [Kubernetes](https://kubernetes.io/)
2. [Terraform](https://www.terraform.io/)
3. [Vault](https://www.vaultproject.io/)

Well...not really, but, these are some of the tools you can use to achieve the above.

##  The How
In order to achieve our goal, we need to first identify the Cloud Provider offerings we would like, establish the base cost then look for ways to cut costs as much as we possibly can.
Given that the Google Cloud Platform offers a generous free tier of products, we shall push our utility of these products to the maximum.

We shall also use a number of open source projects to help us achieve our goal.

## The What

| Product | Purpose | Type | Cost $ | Our Choice | Our Cost $ | Notes |
| :-- | :-- | :--|:-- |:-- |:-- |:-- |
| Google Kubernetes Engine (GKE) | Manage our cluster of machines (nodes) and applications | Regional Cluster | 72.00 | Zonal Cluster |  0.00 |  A regional cluster is encouraged to mitigate against outages in a single zone but costs *72$*. The first zonal cluster is free. We will use a secure private cluster with no public ips and internet access by default. |
| Compute Engine  | These are the actual machines in our cluster. Our applications use resources in these machines. | e2-micro*  | 6.11 | e2-micro pvm* |  1.83 | We use the cheaper pre-emptible machines because GKE will take care of re-creating them when they are terminated |
| Cloud Load Balancer | Route requests from our users to our applications in the cluster. | External | 18.00 | Google Container f1-micro | 0.00 | GCP provides an always free f1-micro instance. We will use this instance together with traefik to route [traefik](https://traefik.io/) into our cluster. |
| Cloud NAT | Access the internet from machines inside our cluster | Google Managed*  | 1.008 | Custom NAT using f1-micro instance | 0.00| GCP provides an always free Public IP. We will use this IP attached to our free f1-micro as the next hop when connecting to the internet from our private cluster  |
| Cloud KMS | :-- | :--|:-- | :--|:-- |:-- |
| Google Secrets Manager | :-- | :--|:-- | :--|:-- |:-- |
| Google Domains | :-- | :--|:-- | :--|:-- |:-- |
| Vault | :-- | :--|:-- | :--|:-- |:-- |
| Cloud Run | :-- | :--|:-- | :--|:-- |:-- |

\* Price indicated is per instance
## Feedback
