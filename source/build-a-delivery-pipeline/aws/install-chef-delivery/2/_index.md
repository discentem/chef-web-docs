## 2. Install Chef Delivery

[CALLOUT networks/admin-workstation.svg] Perform this part from the administrator's workstation.

The installation consists of four steps:

1. Set up a provisioning node or administrator's workstation.
1. Create a configuration file with your desired settings.
1. Run the installation automation to create the cluster.
1. Verify the installation and perform final configuration.

### 2.1. Set up the provisioning node

In order to create the AWS infrastructure, you'll use automation provided by the [delivery-cluster cookbook](https://github.com/chef-cookbooks/delivery-cluster). You'll need a machine to run the provisioning recipes. This can be an administrator's workstation or a server on AWS with SSH access. If you are working through the whole tutorial by yourself, you can use the same workstation as a provisioning node that you'll use later for creating changes in Chef Delivery. No matter which of these options you choose, we'll refer to the machine you use to run the installation automation as the cluster's _provisioning node_.

Choose a machine that will run the automation recipes. It can run any OS that meets these [system requirements](https://docs.chef.io/supported_platforms.html#chef-dk-title). Prepare this machine using the following steps:

#### 2.1.1. Download a temporary Chef Delivery license key

Chef Delivery requires a license key. If you don't have one, click the button below, fill out the short form, and check your email for a link to a temporary license key for this tutorial.

[START_MODAL get-a-license-key Get a license key]

<script src="//app-sj05.marketo.com/js/forms2/js/forms2.min.js"></script>
<form id="mktoForm_1438">
  <h3>Get a Delivery License Key</h3>
</form>
<script>MktoForms2.loadForm("//app-sj05.marketo.com", "255-VFB-268", 1438);</script>

[END_MODAL]

After you click the link in the email, the license file named <code class="file-path">delivery.license</code> will be downloaded to your web browser’s downloads directory. Move this file to your home directory. The final path is <code class="file-path">~/delivery.license</code>.

#### 2.1.2. Install Git

You'll need [Git](https://git-scm.com/downloads) to obtain the automation project that installs Chef Delivery.

[WINDOWS] Many Windows users use Git BASH, which is part of [Git for Windows](https://git-for-windows.github.io). [posh-git](https://github.com/dahlbyk/posh-git) is another popular option, which provides access to Git from Windows PowerShell.

#### 2.1.3. Install the Chef Development Kit

The automation project uses the Chef tools to install Chef Delivery. The easiest way to get these tools is to [install the Chef Development Kit](https://downloads.chef.io/chef-dk/), or Chef DK.

Be sure to set the system Ruby; for details, see [Add Ruby to $PATH](https://docs.chef.io/install_dk.html#add-ruby-to-path).

#### 2.1.4. Install the knife push plugin

The `knife push` plugin enables you to view the status of the build nodes in your Delivery cluster.

```bash
$ chef gem install knife-push
```

#### 2.1.5. Install Rake

The automation project uses [Rake](http://rake.rubyforge.org/) tasks to perform installation tasks. Run this to install it.

```bash
$ chef gem install rake
```

#### 2.1.6. Clone the Git repository for the Chef Delivery cluster

Run this command from your home directory. This will create a local copy of the `delivery-cluster` cookbook on your system. You'll run it later to automate the Chef Delivery installation.

```bash
$ cd ~
$ git clone https://github.com/opscode-cookbooks/delivery-cluster.git
Cloning into 'delivery-cluster'...
remote: Counting objects: 3199, done.
remote: Compressing objects: 100% (6/6), done.
Receiving objects: 100% (3199/3199), 621.11 KiB | 0 bytes/s, done.
remote: Total 3199 (delta 0), reused 0 (delta 0), pack-reused 3193
Resolving deltas: 100% (1921/1921), done.
Checking connectivity... done.
```

#### 2.1.7. Set up your AWS credentials

Place your AWS key credentials in two files, <code class="file-path">~/.aws/config</code> and <code class="file-path">~/.aws/credentials</code> using the text editor of your choice.

For <code class="file-path">~/.aws/config</code>:

```ruby
# ~/.aws/config
[default]
region = us-west-2
output = text
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_KEY
```

Replace <code class="placeholder">us-west-2</code> with your AWS region, and <code class="placeholder">YOUR\_ACCESS\_KEY\_ID</code> and <code class="placeholder">YOUR\_SECRET\_KEY</code> with your access keys.

For <code class="file-path">~/.aws/credentials</code>:

```ruby
# ~/.aws/credentials
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_KEY
```

#### 2.1.8. Set up your AWS SSH private key

Transfer the private key of the key pair that you will use to create the servers in AWS to <code class="file-path">~/.ssh</code> on the provisioning node. If you do not have one already, use this link and create a key pair: [Amazon EC2 Key Pairs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).

### 2.2. Create a configuration file to drive Chef provisioning

The next step is to run a command on the provisioning node that creates an `environments/ENV_NAME.json` file that contains the details of the cluster to be created. This step only creates the configuration file. It doesn't connect to AWS. Run this command:

```bash
$ cd ~/delivery-cluster
$ rake setup:generate_env
```

The command starts the interaction that is shown below. Here are some guidelines:

* Variables you must enter are shown in ALL CAPs.
* Default values are shown in square brackets; pressing Enter uses the default value.
* **Environment Name** specifies the filename that is created. It should be alphanumeric without capital letters, for example, `my-enterprise`.
* For **Driver Name**, specify `aws`.
* **Key Name** specifies the name of your AWS SSH private key, minus any file extension, for example `tpetchel`.
* For **SSH Username**, `ubuntu` is typical for use with Ubuntu; `root` is typical for Red Hat Enterprise Linux.
* **Image ID**, **Subnet ID**, and **Security Group ID** are the IDs for the AWS environment that you chose in the [previous step](#step1).
* For **Number of Build Nodes**, enter a value between 1 and 3. Specifying more than one build node enables phases to be run in parallel.

```bash
# ~/delivery-cluster
Gathering Cluster Information

Provide the following information to generate your environment.

Global Attributes
Environment Name [test]:
Cluster ID [test]:

Available Drivers: [ aws | ssh | vagrant ]
Driver Name [vagrant]: aws

Driver Information [aws]
Key Name: YOUR_KEY_NAME
SSH Username [ubuntu]: YOUR_SSH_USERNAME
Image ID [ami-3d50120d]: YOUR_AMI_ID
Subnet ID [subnet-19ac017c]: YOUR_SUBNET_ID
Security Group ID [sg-cbacf8ae]: YOUR_SECURITY_GROUP_ID
Use private ip for ssh? [yes]:
Would you like to configure Proxy Settings? [no]:

Chef Server
Organization Name [test]:
Use existing chef-server? [no]:
Flavor [c3.xlarge]:

Delivery Server
Package Version [latest]:
Enterprise Name [test]:
License File [/home/tpetchel/delivery.license]:
Flavor [c3.xlarge]:

Analytics Server
Enable Analytics? [no]:

Supermarket Server
Enable Supermarket? [no]:

Build Nodes
Number of Build Nodes [1]:
Flavor [c3.large]:
Specify a delivery-cli artifact? [no]:

Rendering Environment => environments/test.json

{
  "name": "test",
  "description": "Delivery Cluster Environment",
  "json_class": "Chef::Environment",
  "chef_type": "environment",
  "override_attributes": {
    "delivery-cluster": {
      "id": "test",
      "driver": "aws",
      "aws": {
        "key_name": "tpetchel",
        "ssh_username": "ubuntu",
        "image_id": "ami-3d50120d",
        "subnet_id": "subnet-19ac017c",
        "security_group_ids": "sg-cbacf8ae",
        "use_private_ip_for_ssh": true
      },
      "chef-server": {
        "organization": "test",
        "existing": false,
        "flavor": "c3.xlarge"
      },
      "delivery": {
        "version": "latest",
        "enterprise": "test",
        "license_file": "/home/thomaspetchel/delivery.license",
        "flavor": "c3.xlarge"
      },
      "builders": {
        "count": "1",
        "flavor": "c3.large"
      }
    }
  }
}

Export your new environment by executing:
  # export CHEF_ENV=test
```

Next, export the environment by executing the following:

```bash
# ~/delivery-cluster
$ export CHEF_ENV=test
```

where `test` is the name of the environment you provided. You can manage more than one Delivery installation from your workstation. The `CHEF_ENV` environment variable tells `delivery-cluster` which environment to perform installation and maintenance tasks on.

Verify that your <code class="file-path">environments/ENV.json</code> file exists and contains the values you want.

### 2.3. Create the cluster

Now you're ready to create the Chef Delivery cluster. Run the following command:

```bash
# ~/delivery-cluster
$ cd ~/delivery-cluster
$ rake setup:cluster
```

When the `rake setup:cluster` command finishes, you can look in the AWS Console and check that all machines have been created and are running. You'll see one machine for your Chef server, one for Chef Delivery, and one for each build node.

Then, do these steps to check that everything worked:

1. Run the following command and record the output. You will need this information later.

  ```bash
  # ~/delivery-cluster
  $ cd ~/delivery-cluster
  $ rake info:list_core_services
  4 items found

  delivery-server-test:
    ipaddress: 10.194.9.174

  build-node-test-2:
    ipaddress: 10.194.11.102

  build-node-test-3:
    ipaddress: 10.194.12.81

  build-node-test-1:
    ipaddress: 10.194.15.78

  chef_server_url      'https://10.194.13.167/organizations/test'
  ```

1. Navigate to your `chef_server_url` login to your Chef server with user name `delivery` and password `delivery`.

1. Click on **Nodes**. You’ll see your Chef Delivery server and your build nodes.

  ![](delivery/chef-server-nodes-cluster.png)

1. Run the following command:

  ```bash
  # ~/delivery-cluster
  $ rake info:delivery_creds
  Created enterprise: test
  Admin username: admin
  Admin password: GQZUUvJK5xjLdKH5IZyR/QDJE8fzQNlKIuR=
  Builder Password: X+ltuOAZ0kfdZU79p2+sSlGW8sjo+UjJ2ZY=
  Web login: https://10.194.9.174/e/test/
  ```

  The result shows the admin login credentials that were created by the `delivery-cluster` cookbook. Record the output. You will need this information later.

1. Run the following command:

  ```bash
  # ~/delivery-cluster
  $ knife node status
  build-node-test-1	available
  build-node-test-2	available
  build-node-test-3	available
  ```

  All build nodes should report available.

1. Navigate to the Delivery web page (**Web login**) and use the `admin` credentials (**Admin username** and **Admin password**) to log in. You'll see the Delivery dashboard.

  ![](delivery/delivery-dashboard.png)

### 2.4. Finalize the configuration

[CALLOUT networks/delivery-server.svg] Perform this part from the Chef Delivery web interface.

In Chef Delivery there are enterprises, organizations, and projects. The provisioning step created the initial enterprise you specified in your environment file. You still need to finalize the set up by adding adding users and organizations. (The `admin` user is intended only for installation and maintenance.)

Add a user for each team member, or add a single user if you are doing this tutorial independently.

#### 2.4.1. Add a user

Log into the Chef Delivery web UI using the `admin` user. Then, for each user you want to create, do the following:

1. Select **Users** from the drop-down menu on the upper right.
1. Click the plus sign (**+**) next to **Add a New User**.
1. In the Add New a User text area, select **Internal**.
1. Text boxes for first name, last name, email address, a login name and password appear.
  Fill in the text boxes but leave the **SSH Public Key** area blank. The user must log in and enter this information.
1. Select user **Roles Within the Enterprise**. The [documentation](https://docs.chef.io/delivery_users_and_roles.html#roles-and-permissions) describes each available role. Any user who creates a project requires the **Admin** role.
1. Click **Save and Close**, or **Cancel** to discard the operation.

[COMMENT] If you are creating a user on behalf of another person, you can set a temporary password and have that user update it the first time he or she logs in.

#### 2.4.2. Create an organization

Create an organization for the tutorial.

1. Select **Organizations** from the drop-down menu on the upper right. The organizations list page opens.
1. Click the plus sign (**+**) next to **Add a New Organization**.
1. Enter an organization name in the text area, such as `Development`, and then click **Save & Close**.

At this point, Chef Delivery is ready to use for a new project.