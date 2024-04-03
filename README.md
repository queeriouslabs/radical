# Radical
This repo hold ansible configuration for `radical` https://wiki.queeriouslabs.com/radical-machine.

## Testing
A vagrant file is included to assist with building a local VM to test deployment.  It uses a few assumptions:

You many need to change, for example, the ip address to work on your machine.  You will need to address this in `vm.yml` as well.

```ruby
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "generic/debian12"
    config.vm.network "private_network", ip: "192.168.56.105"  # ansible_host
    config.vm.define "radical"

    config.trigger.after :destroy do |trigger|
      trigger.run = {inline: "./remove-key.sh"}
    end


end
```
If you've never used `vagrant`, it's a tool for running other tools, specifically virtualization applications like `libvirt`, `virtualbox`, `qemu` etc.  `vagrant` calls these applications `providers`.  

`vagrant` will assemble a VM based on a provider installed on your machine.  The selection of provider has implicates for the location of the ssh key `vagrant` crates.  For example, if using the provider `virtualbox`, vagrant will create config in a directory `.vagrant/machiens/radical/virtualbox/`. 

Please see, and perhaps update, `vm.yml` to reflect this locaiton to allow ansible to use this key to ssh into the VM:

As the time of this writing, `vm.yml` has atleast 2 items which may require user updates: `ansible_host` to reflect the local IP and `ansible_ssh_private_key_file`.

```yaml
---
all:
  hosts:
    radical:
      ansible_host: 192.168.56.105
      ansible_port: 22
      ansible_user: vagrant
      ansible_ssh_private_key_file: '../../.vagrant/machines/radical/virtualbox/private_key'
      ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
```


> ** Note **
The script `remove-key.sh` runs after vagrant destroy to remove the entry for the VM from your `known_hosts` file.  If not handled, this will cause an error when running a 2nd time.  Note the trigger in the `vagrantfile`.  This WILL (attempt) to modify your `known_hosts` file.  This isn't great practice but you will want this after the 3rd time running a fresh build fails for this reason.


To create a virtual machine to test deployment:

```bash
$ vagrant up
```

This is required to trgansfer the key until I figure out how to automate that

```bash
$ vagrant ssh
```

Then you can deploy via ansible from the `ansible` directory:

```bash
$ cd ansible
$ ansible-playbook -i vm.yml radical.yml
```

When complete, destroy the vm from the same directory as the vagrant file:

```bash
$ vagrant destroy
```

## Deployment
You're going to have to deploy from inside the space to target radical on the router.

From the `ansible` directory

```bash
$ ansible-playbook -i hosts.yml radical.yml 
```
