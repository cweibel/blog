# Wait until SSH is available

Ever run into a situation where you need to ssh into a newly created server but you aren't sure that the server is listening on the ssh port yet?  For the Terraform [OpenStack](https://github.com/cloudfoundry-community/terraform-openstack-cf-install) install of Cloud Foundry the Bastion server isn't immediately available for the provision script to run.

Below is a short bash script that will wait until ssh is available on the target server.  If the server isn't ready yet, wait 10 seconds and try again for a maximum of 10 attempts.


```bash
#Wait until SSH on Bastion server is working
keyPath=$(terraform output key_path | sed -e "s#^~#$HOME#")
scriptPath="provision/provision.sh"
targetPath="/home/ubuntu/provision.sh"
bastionIP=$(terraform output bastion_ip)
maxConnectionAttempts=10
sleepSeconds=10

#Wait until SSH on Bastion server is working
echo "Attempting to SSH to Bastion server..."
index=1

while (( $index <= $maxConnectionAttempts ))
do
  ssh -i ${keyPath} ubuntu@$bastionIP
  case $? in
    (0) echo "${index}> Success"; break ;;
    (*) echo "${index} of ${maxConnectionAttempts}> Bastion SSH server not ready yet, waiting ${sleepSeconds} seconds..." ;;
  esac
  sleep $sleepSeconds
  ((index+=1))
done
```

Enjoy!
