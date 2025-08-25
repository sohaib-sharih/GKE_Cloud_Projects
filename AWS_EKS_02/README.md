### 9.2 AWS Cloud

**Prerequisites**

1. AWS Account
2. installing Kubectl
3. EKS Cluster Role
4. IAM Role for NodeGroup
5. VPS
6. EC2 Key pair which can be used to ssh into worker nodes.

### 9.2.1 Prerequisites of AWS EKS

1. Set up AWS CLI and configure credentials
2. Install eksctl
3. Install kubectl
4. Assign Cluster Role
5. Assign Node IAM role (Identity Access Management Role)

### 9.2.1.1 Installing AWS CLI

1. Install the AWS CLI

```
cd to a folder where you want to download the fil

Example
mkdir awscli
cd awscli

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

OUTPUT:

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
 38 59.1M   38 22.5M    0     0  3177k      0  0:00:19  0:00:07  0:00:12 3901k
 
 
COMMAND INTERPRETATION:

1. curl allows you to download the file from the url.
2. -o -> With a small letter o, allows you to set a custom name for the file.
3. .zip/Name of file: The name of the file, need not have an extension, linux will read the contents of the file and determine automatically if the file is a zipped file
   
```

2. **Unzip the file**

```
unzip awscliv2.zip

NOTE: IF you dont have unzip, then install that first:

sudo apt install unzip -y

unzip awscliv2.zip
```

3. **Install aws**
```
sudo ./aws/install

NOTE: By default, the files are all installed to `/usr/local/aws-cli`, and a symbolic link is created in `/usr/local/bin`.

Check the version of installation

aws --version
 
OUTPUT:

aws-cli/2.28.12 Python/3.13.4 Linux/5.15.153.1-microsoft-standard-WSL2 exe/x86_64.ubuntu.22
```

4. **Delete the installer + zip file**

```
rm awscliv2.zip
```

5. Reference link for installation guide: https://docs.aws.amazon.com/eks/latest/userguide/install-awscli.html
6. **Configure AWS CLI**
	a. Create an ACCESS key, follow the instructions here -> https://docs.aws.amazon.com/eks/latest/userguide/install-awscli.html
	b. Login to your console, click on your top right drop down, then your ***Security Credentials.***
	c. Click on ***Create Key.*** Then download the CSV file.
	d. **AWS CONFIGURE:** Then type `aws configure` command, and fill in the details, and your AWS CLI will be configured.
	e. **Optional:** If you want to set a Security Token that expires after a certain amount of time, you can do that by following the instructions in the link.
7. **Configuring AWS CLI**
```
1. Go to top right corner and select Security Credentials.
2. Create a Key 
3. Under **Access keys**, choose **Create access key**.
4. Choose **Command Line Interface (CLI)**, then choose **Next**.   
5. Choose **Create access key**.
6. Choose **Download .csv file**.
   
```
### 9.2.1.2 Installing AWS + EKS CLI

1. There are 2 methods to install a new version of **Kubectl**
	a. **Overwrite** : You can overwrite the existing kubectl installed in your computer
	b. **Switch between kubectl CLI's**: This method allows you to keep all the different versions of kubectl, and you can easily switch between
2. **NOTE**: If you follow the directions as mentioned in the documentation, it will simply overwrite your existing kubectl version, if you want to keep your current version, and also switch between various different version, you can follow the below steps:

STEPS to installing AWS + EKS(GPT)

| Step                                     | Command/Action                                                                                                                                                                                                                                                                      | Explanation                                                        |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 1. Create a folder for multiple versions | mkdir -p $HOME/kubectl-versions                                                                                                                                                                                                                                                     | Keeps things organized.                                            |
| 2. Download AWS EKS compatible kubectl   | `curl -o $HOME/kubectl-versions/kubectl \ https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2025-08-03/bin/linux/amd64/kubectl`                                                                                                                                                  | Example: downloads AWS EKS `v1.31`. Use correct version from docs. |
| 3. Download GKE/latest kubectl           | `curl -o $HOME/kubectl-versions/kubectl-gke-v1.33 \ https://dl.k8s.io/release/v1.33.3/bin/linux/amd64/kubectl`                                                                                                                                                                      | Example: downloads official latest `v1.33.3`.                      |
| 4. Make binaries executable              | chmod +x $HOME/kubectl-versions/*                                                                                                                                                                                                                                                   | Required to run them.                                              |
| 5. Switch versions manually              | `export PATH=$HOME/kubectl-versions:$PATH` and rename symlink:  <br>`ln -sf $HOME/kubectl-versions/kubectl-eks-v1.31 $HOME/kubectl`                                                                                                                                                 | Lets you choose which kubectl you want.                            |
| 6. Verify version                        | ./kubectl version --client                                                                                                                                                                                                                                                          | Confirms active version.                                           |
| 7. (Optional) Use update-alternatives    | `sudo update-alternatives --install /usr/local/bin/kubectl kubectl $HOME/kubectl-versions/kubectl-eks-v1.31 1`  <br>`sudo update-alternatives --install /usr/local/bin/kubectl kubectl $HOME/kubectl-versions/kubectl-gke-v1.33 2`  <br>`sudo update-alternatives --config kubectl` | Provides an interactive menu to switch between versions easily.    |

### Interpreting the installation curl command

1. Two ways to **download** and **save** the kubectl cli
	a. Saving using -O
	b. Saving using -o

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.3/2025-08-03/bin/linux/amd64/kubectl

INTERPRETATION
1. `-O` (capital O) → tells `curl` to save the file with its original name as given in the URL.
2. Here, the file in the URL ends with `kubectl`, so it will be saved as `kubectl` in your current working directory.
   
---------------------------
curl -o mykubectl https://.../kubectl
OR
curl -o $HOME/kubectl-versions/kubectl-eks-v1.31 \
https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2025-08-03/bin/linux/amd64/kubectl



INTERPRETATION:
1. `-o <filename>` (small o) → lets you specify a **custom filename** for the downloaded file.
2. This saves the file as `mykubectl` instead of `kubectl`.
3. `-o $HOME/kubectl-versions/kubectl-eks-v1.31` → this explicitly sets the filename where the binary will be saved (`kubectl-eks-v1.31`).
4. `kubectl-eks-v1.31` is the filename I(gpt) chose in the path. That’s what lets you keep multiple versions instead of overwriting.
```

### Making the file Executable

1. **Apply** execute permissions to the binary

```
chmod +x ./kubectl
```

2. Copy the binary to a folder in your `PATH`. If you have already installed a version of `kubectl`, then we recommend creating a `$HOME/bin/kubectl` and ensuring that `$HOME/bin` comes first in your `$PATH`.

```
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
```

INTERPRETATION

| Part                           | Meaning                                                                                | What it does                                                                                                                        |
| ------------------------------ | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| mkdir -p $HOME/bin             | `mkdir` = make directory. `-p` = create parent dirs if not exist (no error if exists). | Creates a folder `bin` inside your home directory if it doesn’t already exist.                                                      |
| &&                             | Logical AND operator in bash.                                                          | Runs the next command **only if** the previous one succeeds.                                                                        |
| cp ./kubectl $HOME/bin/kubectl | `cp` = copy.                                                                           | Copies the `kubectl` file from the current directory (`./`) into `$HOME/bin/`. It **does not remove** the original (not like `mv`). |
| export PATH=$HOME/bin:$PATH    | Update `PATH` env variable.                                                            | Ensures `$HOME/bin` comes **first**, so this `kubectl` version is found before others.                                              |
### Difference between Programs with Installers VS Binaries (gpt)

| AWS CLI (Installer based)                                                                                                                                                | Kubectl EKS Cli (Standalone Binary)                                                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **AWS CLI** → Installed via an installer, which places it under `/usr/local/bin` (a system-wide location in `$PATH`). That makes it **globally available** to all users. | **kubectl (from EKS docs)** → Often downloaded manually as a **standalone binary**. If you put it in `$HOME/bin` or `$HOME/kubectl-versions`, it’s only available for **your user**, unless you add that path to your `$PATH`. |
| **Installer-based programs** → go to system-wide dirs (`/usr/bin`, `/usr/local/bin`).                                                                                    | **Manually downloaded binaries** → you decide where to keep them (`$HOME/...`), and only you can access them unless you move them to a global dir.                                                                             |
### Path Variable Priority (gpt)

1. When you set a PATH variable, you must define the previously set paths and ***APPEND*** the new one using the ***colon*** symbol. *Otherwise it will erase the previous paths defined in the PATH variable, and the new one will overwrite.*
2. `mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH` --> This command puts your new `kubectl` in `$HOME/bin`, then updates `$PATH` so `$HOME/bin` is searched **first**, making that `kubectl` take priority every time.

### Switching between different versions of the CLI (gpt)

**Method 1: Symbolic link (simpler)**

1. Put all versions in `$HOME/kubectl-versions/`.  Example:
	a. kubectl-gke-v1.33
	b. kubectl-eks-v1.30
2. Create/update a symlink in `$HOME/bin`:
```
ln -sf $HOME/kubectl-versions/kubectl-gke-v1.33 $HOME/bin/kubectl

NOTE -> Now `kubectl` points to v1.33.
```
3. To switch: just change the symlink:
```
ln -sf $HOME/kubectl-versions/kubectl-eks-v1.30 $HOME/bin/kubectl

```

**Method 2: `update-alternatives`**

1. Register versions:
```
sudo update-alternatives --install /usr/local/bin/kubectl kubectl $HOME/kubectl-versions/kubectl-gke-v1.33 10
sudo update-alternatives --install /usr/local/bin/kubectl kubectl $HOME/kubectl-versions/kubectl-eks-v1.30 20

```
2. Switch with menu:
```
sudo update-alternatives --config kubectl

OUTPUT EXAMPLE:

There are 2 choices for the alternative kubectl (providing /usr/local/bin/kubectl).

  Selection    Path                                          Priority   Status
------------------------------------------------------------
* 0            /home/user/kubectl-versions/kubectl-gke-v1.33 10        auto mode
  1            /home/user/kubectl-versions/kubectl-gke-v1.33 10        manual mode
  2            /home/user/kubectl-versions/kubectl-eks-v1.30 20        manual mode

NOTE:
1. Then type `1` or `2` to switch.

```
3. Which is better?
	a. If you’re learning/testing → **symlink method** (faster & no sudo).
    b. If you want a clean system-wide solution → **update-alternatives**.

4. **SYMBOLIC LINK:** Do you have to use the **SYMBOLIC LINK?**
	a. You don't have to if you have a kubectl version installed at root level and just 1 installed in another directory/path.
	b. You can simply change the **PRIORITY** of the path, it will act as ***switching*** between, but in reality it will just be changing the priority, which means that when you run ***kubectl*** command, the shell will search for it in the PATH defined as ***order of sequence***
	c. Example
```
export PATH="$HOME/kubectl-versions:$PATH"

INTERPRETATION
1. It will search for it at $HOME/kubectl-versions/ directory first, if it finds it then it will run using that binary.
```


### SUMMARY OF INSTALLING AWS -EKS

```
mkdir $HOME/kubectl-versions

curl -o $HOME/kubectl-versions/kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.3/2025-08-03/bin/linux/amd64/kubectl

chmod +x #HOME/kubectl-versions/kubectl

echo 'export PATH="$HOME/kubectl-versions/:$PATH"' >> ~/.bashrc

source ~/.bashrc

kubectl version --client

OUTPUT:
Client Version: v1.33.3-eks-3abbec1
Kustomize Version: v5.6.0


-----------------------------

HOW TO FIX SOME COMMON ERRORS
1. mv $HOME/kubectl-versions/kubectl-eks-v1.33 $HOME/kubectl-versions/kubectl (This renames the binary file)
2. The binary file name should be kubectl because that is what you will use to run the command as kubectl.
3. curl custom path and name of file must be defined correctly and should be kubectl as shown above.
```


### 9.2.1.3 Assigning Roles

### EKS Cluster Role Creation

1. **AWS Management Console:** Open the IAM console at https://console.aws.amazon.com/iam/.
2. Choose **Roles**, then **Create role**.
3. Under **Trusted entity type**, select **AWS service**.
4. From the **Use cases for other AWS services** dropdown list, choose **EKS**.
5. Choose **EKS - Cluster** for your use case, and then choose **Next**.
6. On the **Add permissions** tab, choose **Next**.
7. For **Role name**, enter a unique name for your role, such as `eksClusterRole`.
8. For **Description**, enter descriptive text such as `Amazon EKS - Cluster role`.
9. Choose **Create role**.
10. **Reference Link for documentation(instruction)->** https://docs.aws.amazon.com/eks/latest/userguide/cluster-iam-role.html#create-service-role
```
1. Create the role. You can replace `eksClusterRole` with any name that you choose.
2. You need to copy and paste the following in a json file called 'cluster-trust-policy.json'
------------The below json content------------------------
   {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

3. Then run the following command while the file is in the current directory.

aws iam create-role \
  --role-name eksClusterRole \
  --assume-role-policy-document file://"cluster-trust-policy.json"
```

11. *Why is this required?* (Why do we copy & paste if the content is the same)
	a. AWS gives you **templates** (pre-defined JSON snippets like "allow sts:TagSession").
    b. But those rules don’t apply to _anyone_ until you **attach them** to a specific role, user, or group.

12. *Why they become unique:*
	a. The _content_ of the policy may look the same, but the **attachment makes it unique** because it is now bound to _your_ role (`eksClusterRole`), not someone else’s.
    b. AWS looks at the **role identity + attached policies** together to decide what that entity can do.
    c. ***Example:*** two people can both carry the same rulebook, but only the one who has it _in their pocket_ can follow those rules.

So: policies themselves aren’t unique, but **the binding (attachment) to your role/user is what makes permissions unique to you.**

13. After ***ROLE CREATION***, you must ***ATTACH THE POLICY***
	a. Copy and Paste the policy into a ***json*** file called ***AmazonEKSClusterPolicy.json***
	b. Attach the required IAM policy to the role: 

**COMMAND TO ATTACH**

```
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy --role-name eksClusterRole
```

**AmazonEKSClusterPolicy.json file**

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AmazonEKSClusterPolicy",
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:UpdateAutoScalingGroup",
                "ec2:AttachVolume",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CreateRoute",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:DeleteRoute",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteVolume",
                "ec2:DescribeInstances",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVolumes",
                "ec2:DescribeVolumesModifications",
                "ec2:DescribeVpcs",
                "ec2:DescribeDhcpOptions",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DescribeAvailabilityZones",
                "ec2:DetachVolume",
                "ec2:ModifyInstanceAttribute",
                "ec2:ModifyVolume",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:DescribeAccountAttributes",
                "ec2:DescribeAddresses",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeInstanceTopology",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                "elasticloadbalancing:AttachLoadBalancerToSubnets",
                "elasticloadbalancing:ConfigureHealthCheck",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancerListeners",
                "elasticloadbalancing:CreateLoadBalancerPolicy",
                "elasticloadbalancing:CreateTargetGroup",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:DeleteLoadBalancer",
                "elasticloadbalancing:DeleteLoadBalancerListeners",
                "elasticloadbalancing:DeleteTargetGroup",
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:DeregisterTargets",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "elasticloadbalancing:DescribeLoadBalancerPolicies",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeTargetGroupAttributes",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:DetachLoadBalancerFromSubnets",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:ModifyLoadBalancerAttributes",
                "elasticloadbalancing:ModifyTargetGroup",
                "elasticloadbalancing:ModifyTargetGroupAttributes",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AmazonEKSClusterPolicySLRCreate",
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
                }
            }
        },
        {
            "Sid": "AmazonEKSClusterPolicyENIDelete",
            "Effect": "Allow",
            "Action": "ec2:DeleteNetworkInterface",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "ec2:ResourceTag/eks:eni:owner": "amazon-vpc-cni"
                }
            }
        }
    ]
}
```


### Cluster Role Policy

1. Do we need to ***Copy and paste*** the json file in your local directory to attach policies? ***NO, its not required. When you create a role, you must tell AWS 'who can assume it' (EKS, EC2, etc.).***
2. **Types of Policies:**
	a. Trust Policy (Assume Role Policy)
	b. Permissions Policies (like AmazonEKSClusterPolicy)
3. **Trust policy JSON** = required (only once, at role creation).
4. **AWS-managed policies** = already in AWS, just attach by ARN (no JSON files needed).
5. Required policies for **EKS Auto Mode** cluster role:
	a.  `AmazonEKSClusterPolicy`
	b. `AmazonEKSBlockStoragePolicy`
    c. `AmazonEKSComputePolicy`
    d. `AmazonEKSLoadBalancingPolicy`
    e. `AmazonEKSNetworkingPolicy`

```
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSBlockStoragePolicy --role-name eksClusterRole

-------------------

aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSComputePolicy --role-name eksClusterRole

------------------------

aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSLoadBalancingPolicy --role-name eksClusterRole

-------------------------
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSNetworkingPolicy --role-name eksClusterRole 

```

### Adding Tag Error on Cluster Role Selection(While creating a new eks cluster)

1. **Error message in the console:**
```
The cluster role must have the following actions specified in its trust policy to use EKS Auto Mode: sts:TagSession
```

2. **Solution:**

```
Change the following cluster-trust-policy.json

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

---------------------
to the below json

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": [
        "sts:AssumeRole",
        "sts:TagSession"
      ]
    }
  ]
}

-------------------------
Apply the following command

aws iam update-assume-role-policy \
  --role-name eksClusterRole \
  --policy-document file://"cluster-trust-policy.json"

1. `create-role` uses `--assume-role-policy-document` (because you’re attaching it for the first time).
2. `update-assume-role-policy` uses `--policy-document` (because you’re replacing the existing one).

NOTE: Therefore you use this command to only UPDATE the previously created cluster role.
```

3. **Why are tags used?**
	a. **Tags (key–value pairs in the Tags tab):** Just metadata for organizing/billing. They don’t give or restrict permissions.
	b. **Cluster trust policy (JSON with `sts:AssumeRole` etc.):** Defines which AWS service (like EKS) can assume the role. If AWS is saying you must add `sts:TagSession`, then you **must update the trust policy JSON** — tags alone won’t fix that.
	c. So you can add tags for organization if needed, but **you still need to edit and reapply the trust policy** with the required actions.
	d. AWS **tags** are just **key–value labels** you can attach to resources (e.g., `env=dev`, `owner=hunarmund`).
	e. EKS Auto Mode uses **session tags** internally (when EKS assumes the role) to track and manage cluster resources.
	f. That’s why your **trust policy** must allow `sts:TagSession`.

### Amazon EKS Node IAM role

1. Reference link: https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html
2. Create the role and give it a name following the instructions.
3. Create a json file called -> node-role-trust-relationship.json in the current directory. *Copy and paste the content from the documentation link above.*
4. Create the IAM role and attach 4 policies:
	a. node-role-trust-relationship.json
	b. AmazonEKSWorkerNodePolicy
	c. AmazonEC2ContainerRegistryPullOnly
	d. vpc-cni-trust-policy.json (AmazonEKS_CNI_Policy)
```
CREATE IAM Role

aws iam create-role \ --role-name node_role \ --assume-role-policy-document file://"node-role-trust-relationship.json"

-----------------
The json file that needs to be created in your local repository:

node-role-trust-relationship.json file->

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com"
                ]
            }
        }
    ]
}

NOTE: That JSON is only for creating the role.

---------------------
CHECK IF THE ROLE WAS CREATED:

aws iam get-role --role-name node_role

OUTPUT:
list all the policies attached to the role, but it wont list node-role-trust-relationship.json, because it is saved in AWS, and is hidden.

---------------

If the role already exists, just apply the policies as shown below and on step 5:

aws iam attach-role-policy --role-name node_role --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

aws iam attach-role-policy --role-name node_role --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy

```

5. Attach two required IAM managed policies to the IAM role.

```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
  --role-name node_role
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPullOnly \
  --role-name node_role
  
-------------------------------------

OR run all three if the role already exists:

aws iam attach-role-policy --role-name node_role --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

aws iam attach-role-policy --role-name node_role --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
-------------------------
TO CHECK ALL POLICIES

aws iam list-attached-role-policies --role-name node_role


```

6. **Install eksctl (so you can get the cluster's OIDC's provider URL using EKS):** 
*Reference link for installation:* https://eksctl.io/installation/
	a. Your cluster **doesn’t have an IAM OIDC provider associated** yet. EKS **requires OIDC** to let service accounts assume IAM roles. Without it, `eksctl create iamserviceaccount` will always fail.
	b. **OIDC** = OpenID Connect → an identity layer on top of OAuth 2.0, used by EKS to let Kubernetes service accounts assume AWS IAM roles securely.
	
7. **Purpose of roles and accounts in AWS:**
	a. **Node role** → lets EC2 worker nodes talk to EKS + pull images from ECR. You can’t run pods without it.
	b. **Service account + OIDC** → only needed if a pod needs AWS access (e.g. S3, DynamoDB). For your voting app, not required unless it talks to AWS services.
	c. **Cluster role** → lets EKS control plane manage resources.
	d. Minikube/Docker don’t need IAM because they run locally, AWS needs IAM to connect services securely.



```
https://eksctl.io/installation/

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz"
--------------
tar -xzf eksctl_$(uname -s)_amd64.tar.gz
----------------
sudo mv eksctl /usr/local/bin/

------------
eksctl version

OUTPUT:
0.212.0
```
6. **View your cluster’s OIDC provider URL.**

```
aws eks describe-cluster --name example-voting-app --query "cluster.identity.oidc.issuer" --output text

OUTPUT:

https://oidc.eks.us-east-1.amazonaws.com/id/8A772EC8E00832D499031EB5464789D3

-------------------
Create a json file

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com",
                    "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:kube-system:aws-node"
                }
            }
        }
    ]
}

-------------------------------
Replace the Values of the json file with your
1. Account ID
2. OIDC url output
   
See the changes below in JSON file:

vpc-cni-trust-policy.json file->
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::154292416837:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/8A772EC8E00832D499031EB5464789D3"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.us-east-1.amazonaws.com/id/8A772EC8E00832D499031EB5464789D3:aud": "sts.amazonaws.com",
                    "oidc.eks.us-east-1.amazonaws.com/id/8A772EC8E00832D499031EB5464789D3:sub": "system:serviceaccount:kube-system:aws-node"
                }
            }
        }
    ]
}
----------------
CREATING THE ROLE IF YOU HAVEN'T CREATED IT YET:

aws iam create-role \ --role-name node_role \ --assume-role-policy-document file://"vpc-cni-trust-policy.json"


OUTPUT:
If user role exists(error) then update it


aws iam update-assume-role-policy --role-name node_role --policy-document file://vpc-cni-trust-policy.json

----------------------

Attach the IPv4 Policy

aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy --role-name node_role




```


Rough

```
eksctl create iamserviceaccount \
    --name aws-node \
    --namespace kube-system \
    --cluster my-cluster \
    --role-name AmazonEKSVPCCNIRole \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
    --override-existing-serviceaccounts \
    --approve

```

7. SUMMARY till here:
	a. CREAT the **IAM role** using the ***node-trust-relationship-policy.json*** file
	b. Attach two required IAM managed policies to the IAM role.
	b. Attach the IAM policy (AmazonEKS CNI Policy) to the **IAM role** (This creates the IAM role)
	b. Ref link: https://docs.aws.amazon.com/eks/latest/userguide/cni-iam-role.html
	c. Install EKS so you can get the OIDC Url output
	d. Create a json file -> *vpc-cni-trust-policy.json*
	e. Attach this json file to the ***Existing role*** or a create a **new role**
	f. 

```
eksctl create iamserviceaccount \ --name aws-node \ --namespace kube-system \ --cluster my-cluster \ --role-name AmazonEKSVPCCNIRole \ --attach-policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \ --override-existing-serviceaccounts \ --approve

OUTPUT
Error

------------------------



```


### 9.3 AWS Cloud Setup 2

### Caution

1. If you created the AWS Cloud setup using the methods mentioned in section 9.2, then it is likely that once you ***delete the cluster***, it will also delete the IAM roles and Group you created through EKSCTL CLI.
2. Below is the difference between the formation of a cloud cluster using AWS CLI VS EKSCTL

| Aspect               | AWS CLI                                                                                 | eksctl CLI                                                                                   |
| -------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Purpose              | General tool to manage _all_ AWS services (EC2, S3, IAM, EKS, etc.)                     | specialized tool only for creating and managing **EKS clusters** and related resources       |
| Scope                | Works at low level (you configure everything manually)                                  | High-level abstraction (automates cluster setup)                                             |
| Resource Creation    | You create each resource separately (VPC, IAM roles, nodegroups, cluster, etc.)         | Creates everything for you in one go (cluster, nodegroups, IAM, VPC if needed)               |
| Control              | Full control, you decide exactly what to create and keep                                | Limited control, defaults are chosen by eksctl (but configurable via YAML)                   |
| Deletion Behavior    | Only deletes what you explicitly ask (your manually created roles/policies stay intact) | Deletes entire CloudFormation stacks (removes IAM roles, nodegroups, etc. created by eksctl) |
| Use Case             | When you want reusable, fine-grained, long-term AWS resources                           | When you want to quickly spin up and delete clusters for testing/dev                         |
| Underlying Mechanism | Direct API calls to AWS services                                                        | Uses AWS APIs **+ CloudFormation stacks** to automate infrastructure                         |
| Learning Curve       | Harder, because you must know all the AWS details (VPC, IAM, networking, etc.)          | Easier, since eksctl hides most AWS complexity                                               |
| Flexibility          | Very flexible, works for anything AWS                                                   | Limited to EKS operations (clusters, nodegroups, IAM service accounts, Fargate profiles)     |
**Technical Difference Conclusion:**

1. **AWS CLI** = direct interface to AWS API → you manage resources yourself.
2. **eksctl** = wrapper built on AWS APIs + CloudFormation → automates common EKS patterns but also deletes everything it created when you tear down.

### 9.3.1 AWS Cluster Setup

1. **Reference link for documentation:** https://docs.aws.amazon.com/eks/latest/userguide/automode-get-started-cli.html
2. **Install AWS**: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
3. **Configure AWS:** https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-configure-quickstart-config
4. You may follow the instructions of how to install ***AWS CLI*** and ***EKSCTL*** from the section 9.2.
5. Follow the instructions in sequence below:
	a. https://docs.aws.amazon.com/eks/latest/userguide/automode-get-started-cli.html
	b. You can create the node IAM role with the **AWS Management Console** -> https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html
6. **Few Commands to configure AWS CLI to your system**

```
aws configure

Note: 
1. This will prompt you to enter password and keys
2. To get the keys, go to Credentials section on your aws console
3. Create Key
4. Download the excel document, and it will contain the keys.
5. Will look like the following:

Check your current configuration by:
aws sts get-caller-identity

OUTPUT:
{
    "UserId": "154292416837",
    "Account": "154292416837",
    "Arn": "arn:aws:iam::154292416837:root"
}

--------------
aws configure
OUTPUT:
Will ask you to enter keys and format of the configuration, select or type JSON.
```

7. Once the node IAM role is created (2 methods: Console OR aws CLI), then create the cluster, it can take about 5 minutes to created.
8. **NodeGroup:** Creating the node group is important, it requires a node role.
9. **Configure your computer to communicate with the cluster:** *Ref link: https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html*
```
aws eks update-kubeconfig --region region-code --name my-cluster

cts/AWS_EKS_02$ aws eks update-kubeconfig --region us-east-1 --name example-voting-app

OUTPUT:
Updated context arn:aws:eks:us-east-1:154292416837:cluster/example-voting-app in /home/sohaibsharih/.kube/config

-------------------------
AWS_EKS_02$ kubectl get svc

OUTPUT:
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   44m

----------------------------------
AWS_EKS_02$ kubectl get nodes

OUTPUT:
NAME                  STATUS   ROLES    AGE   VERSION
i-01392e25feb7b53fd   Ready    <none>   34m   v1.33.1-eks-f5be8fb
i-0913085bf37c5c435   Ready    <none>   34m   v1.33.1-eks-f5be8fb
```

10. **Masternodes:** These nodes are managed by the service provider and cannot be accessed by anyone through ssh.