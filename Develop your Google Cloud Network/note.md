
## Multiple VPC NetWork

[Set up and Configure a Cloud Environment in Google Cloud - Multiple VPC Networks | Google Cloud Skills Boost](https://www.cloudskillsboost.google/course_templates/625/labs/566601)

### **Create the privatenet network**

```shell
# create privatenet 
gcloud compute networks create privatenet --subnet-mode=custom

# create privatesubnet-1
gcloud compute networks subnets create privatesubnet-1 --network=privatenet --region=us-west1 --range=172.16.0.0/24

# create privatesubnet-2
gcloud compute networks subnets create privatesubnet-2 --network=privatenet --region=europe-west4 --range=172.20.0.0/20

# list the available VPC subnets (sorted by VPC network):
gcloud compute networks subnets list --sort-by=NETWORK
```

### Create the firewall rules

```shell
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
```

### Create VM with privatenet network

```shell
gcloud compute instances create privatenet-vm-1 --zone=us-west1-a --machine-type=e2-micro --subnet=privatesubnet-1

```

### Explore the connetivity between VM instance

```shell
# check the connetivity from vm1 to vm2
ping -c 3 'ip address'
```

### Create vm with multiple network

```shell
gcloud compute instances create vm-appliance \
    --project=qwiklabs-gcp-00-21e3843e2c99 \
    --zone=us-west1-a \
    --machine-type=e2-standard-4 \
    --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=privatesubnet-1 \
    --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=managementsubnet-1 \
    --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=mynetwork \
    --metadata=enable-osconfig=TRUE,enable-oslogin=true \
    --maintenance-policy=MIGRATE \
    --provisioning-model=STANDARD \
    --service-account=709601049229-compute@developer.gserviceaccount.com \
    --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/trace.append \
    --create-disk=auto-delete=yes,boot=yes,device-name=vm-appliance,disk-resource-policy=projects/qwiklabs-gcp-00-21e3843e2c99/regions/us-west1/resourcePolicies/default-schedule-1,image=projects/debian-cloud/global/images/debian-12-bookworm-v20250709,mode=rw,size=10,type=pd-balanced \
    --no-shielded-secure-boot \
    --shielded-vtpm \
    --shielded-integrity-monitoring \
    --labels=goog-ops-agent-policy=v2-x86-template-1-4-0,goog-ec-src=vm_add-gcloud \
    --reservation-affinity=any \
&& \
printf 'agentsRule:\n  packageState: installed\n  version: latest\ninstanceFilter:\n  inclusionLabels:\n  - labels:\n      goog-ops-agent-policy: v2-x86-template-1-4-0\n' > config.yaml \
&& \
gcloud compute instances ops-agents policies create goog-ops-agent-v2-x86-template-1-4-0-us-west1-a \
    --project=qwiklabs-gcp-00-21e3843e2c99 \
    --zone=us-west1-a \
    --file=config.yaml
```
