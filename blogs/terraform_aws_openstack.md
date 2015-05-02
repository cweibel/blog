# Updates to terraform-aws-cf-install and terraform-openstack-cf-install

There have been a couple exciting changes to the two Cloud Foundry provisioning projects for (AWS) and (OpenStack) which make deploying Cloud Foundry to these two infrastructures much simpler.

See (link) for instructions on deploying Cloud Foundry to AWS, one for OpenStack is coming soon so check the blog again soon.

## make provision
First, the `provision.sh` script is no longer executed in `make apply`. Why? Under the old code if an error was encountered in order to rerun the script the Bastion server would be destroyed, a new one created and the script copied to the new server and executed. If you had work on the Bastion server, say a bunch of custom boshworkspaces, you would lose all this work.

`make apply` now is responsible for setting up the networking and an initial Bastion VM. So the order of commands to stand up Cloud Foundry now is:

```bash
make plan
make apply
make provision
```

The `make apply` must complete successfully before executing `make provision`.  The AWS project requires the `make apply` to be run twice.

`make provision` creates a `provision.sh` script in the `provision/` folder which is copied to the `/home/ubuntu` folder Bastion server.  The script no longer require all 21 command line parameters as they are now automatically populated.  What this means is there is a complete copy of the provision script on your local computer and the Bastion server.

A couple notes
 - Every time you run `make provision` a new copy of `provision.sh` is generated locally, then copied to the Bastion server and finally executed on the the Bastion server.
 - On some OpenStack deployments the Bastion server takes a few seconds before the ssh port 22 is ready to accept connections.  If you run into this, just wait a minute and execute `make provision` again.

## Idempotent provision.sh
We've made the provision script (mostly) able to run as many times as needed to get to a successful state.  This is helpful if RVM, Github or the package manager are having a bad day with some sort of temporary issue, you can simply rerun the provision.sh script and the missing pieces will be installed.

There are a couple known exceptions if the script is terminated that manual intervention is still needed:
 - Termination while running `bosh bootstrap deploy` occurs.  Depending on where it died you may just be able to rerun the script.
 - Termination when packages or jobs are being rebuilt.  You need to wait until there are no BOSH tasks running before running the script again.  You can ssh onto the Bastion server (see the next section on how to easily do this) and run `bosh tasks` to get a list of running BOSH tasks.  

## make ssh
We've added an easy way to ssh to the Bastion server.  As long as `make apply` was successful you can run 'make ssh' and open a ssh connection to the Bastion server.

OSX and Ubuntu/Debian are supported with this feature.

## Terraform v0.4.2

Both projects now work with Terraform v0.4.2!
