# Wait until SSH is available

Ever run into a situation where you need to ssh into a newly created server but you aren't sure that the server is listening on the ssh port yet?  For the Terraform [OpenStack](https://github.com/cloudfoundry-community/terraform-openstack-cf-install) install of Cloud Foundry the Bastion server isn't immediately available for the provision script to run.

Below is a short bash script that will wait until ssh is available on the target server.  If the server isn't ready yet, wait 5 seconds and try again.


```bash
#Wait until SSH on Bastion server is working
bastionIP=$(terraform output bastion_ip)
keyPath=$(terraform output key_path | sed -e "s#^~#$HOME#")
index=1
while (( $index != 0 ))
do
ssh -i ${keyPath} ubuntu@$bastionIP
  case $? in
    (0) echo "${index}> Success"; break ;;
    (*) echo "${index}> Bastion SSH server not ready yet..." ;;
  esac
  sleep 5
  ((index+=1))
done
```

Enjoy!
