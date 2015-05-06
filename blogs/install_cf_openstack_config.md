# Configuring OpenStack for Cloud Foundry

Assumptions:

 - OpenStack is already installed
 - If OpenStack Juno is used, make sure [this patch](https://blog.starkandwayne.com/2015/05/05/openstack-juno-static-ip-patch/) is applied
 - You are provided an admin login or an administrator follows the steps in 'Create A New User' and provides you the credentials
 - An external network has been created
 - An Ubuntu image is already uploaded

We will:

 - Create a new user name and password
 - Create a new Project
 - Retrieve the tenant name
 - Retrieve the tenant id
 - Retrieve the network
 - Retrieve the auth_url
 - Retrieve the floating IP pool
 - Retrieve the external network id
 - Identify a local set of keys to use


## Step 1 - Create a new user name and password

Once you login into Horizon, select the `admin` project, navigate it Identity > Users and click `+ Create User`
![](https://raw.githubusercontent.com/cweibel/blog/master/images/Users-OpenStack-Dashboard-1.jpg)

On the Create User screen, enter a **username** and **password**, set both the Primary Project and Role to `admin` and click `Create User`
![](https://raw.githubusercontent.com/cweibel/blog/master/images/Users-OpenStack-Dashboard-2.png)


## Step 2 - Create a new Project

Select the `admin` project, navigate it Identity > Projects and click `+ Create Project`
![](https://raw.githubusercontent.com/cweibel/blog/master/images/Projects-OpenStack-Dashboard-4.png)

On the `Project Information` tab enter the name for the project. Make sure that the project name does not include white space, this will eventually be fixed. Note that this is also the **tenant name**.
![](https://raw.githubusercontent.com/cweibel/blog/master/images/Projects-OpenStack-Dashboard-3.png)

On the `Project Members` tab, select the user we created back in Step 1 by clicking on the blue "+" icon.
![](https://raw.githubusercontent.com/cweibel/blog/master/images/Projects-OpenStack-Dashboard-2.png)

Once the user is selected, be sure that the permissions for the user are `_member_`, higher permissions are not required.  Click `Create Project` when you are done adding users
![](https://raw.githubusercontent.com/cweibel/blog/master/images/Projects-OpenStack-Dashboard-1.png)


## Step 3 - Retrieve the tenant id

Now logout and login as the user `cf-user` (or whatever user you created in Step 1).

Select `cf-terraform` as the project and navigate to Identity > Projects.  Identify the row which corresponds to the project we created in Step 2. The **tenant id** is located in the Project ID column for the identified project.
![](https://raw.githubusercontent.com/cweibel/blog/master/images/Projects-OpenStack-Dashboard.png)


## Step 4 - Retrieve the network
## Step 5 - Retrieve the auth_url
## Step 6 - Retrieve the floating IP pool
## Step 7 - Retrieve the external network id
## Step 8 - Identify a local set of keys to use

At the end we have the following information:
```
network = "192.168"                                        # Step 4
auth_url="http://10.2.95.20:5000/v2.0"                     # Step 5
tenant_name="terraform_cf"                                 # Step 2
tenant_id="69f43a6776344a338b9cafdea088aca4"               # Step 3
username="cf_user"                                         # Step 2
password="cf_user"                                         # Step 2
public_key_path="/Users/chris/.ssh/id_rsa.pub"             # Step 8
key_path="/Users/chris/.ssh/id_rsa"                        # Step 8
floating_ip_pool="net04_ext"                               # Step 6
region="RegionOne"                                         # TODO is this somewhere?
network_external_id="bb89edee-e639-43b9-b27e-db2fcab833b2" # Step 7
```
