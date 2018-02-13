Local openshift cluster

## Setup
1 Master node
3 Worker node

all are currently scheduleable. it should be better to have a dedicated node as router, but to do this, Apple should deliver 32GB macbooks. 

## How to run this shizzle

1. vagrant plugin install landrush vagrant-hostmanager
2. vagrant landrush start
3. git clone https://github.com/openshift/openshift-ansible
4. cd openshift-ansible && git checkout release-3.7 && cd ..
5. vagrant up
6. ansible-playbook openshift-ansible/playbooks/byo/config.yml
7. wait, drink coffee, wait some more

## WTF you broke my vagrant destroy -f
Yes I did, this to maximize the surviveability of the local registry mirror
You can destroy by adding the machine names

### there is no such thing as too many disks ;-)


sudo killall -HUP mDNSResponder
