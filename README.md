A handy configuration method to setup a workspace/development container with LXD using Ansible. Includes a method to mount a directory from the host with the correct permissions for reading and writing.

## Outline

```
├── ansible.cfg
├── config.yml
├── inventory.ini
├── main.yml
└── tasks
    ├── dotfiles.yml # pulls your dotfiles to make your container feel more like home
    ├── lxd.yml      # creates the container
    ├── mounts.yml   # mount any host directories
    ├── packages.yml # install any needed packages
    └── post.yml     # run any needed commands after container creation
```

## Setup

Note which parts you want to include/exclude by setting them to false or true (enable_mounts, use_dotfiles, post_setup, and install_packages), and  add your host(s) to the ```ansible.cfg``` file.

Commands can be run in the post.yml using ```lxc exec``` through ansible's command module. Commands placed in the ```config.yml``` file.

Note: containers are set to ephemeral by default. This can be changed in the config.yml by setting is_ephemeral to ```no```.

### Run

```
cd /to/the/playbook/directory

# local
ansible-playbook main.yml --connection=local

# none local
ansible-playbook main.yml
```

## Note on SSH

Ansible communicates with ssh. However, the LXD/LXC containers only allow for key authentication by default, so the ```.ssh/authorized_keys``` file will need to be included in the container somehow. However, some images do not come with openssh-server installed by default. To get around this, commands will be issued using ansible's built in ```command``` for post container setup.

```lxc push``` can be used in the 'post.yml' tasks send over the authorized_keys, though the correct ownerships/permissions will also need to be applied after the push is run. That can also be taken care of in the ```post.yml``` task.

As an alternative to pushing files, the default images for Ubuntu (from the source: https://cloud-images.ubuntu.com/releases) have a very convienet mechanism built into LXD. You can use the key servers from either GitHub or Launchpad and have the authoorized_keys added during container creation using the method outlined in [this](https://ubuntu.com/blog/howto-automatically-import-your-public-ssh-keys-into-lxd-instances) blog post. This means for everycontainer created using this profile, you can autmotaclly ssh into it without issue.

Basically, use ```lxc profile edit [profile-name-here]``` to edit your prefered profile and add your GitHub or Launchapd info as 'user.vender-data':

```
# To add authorized_keys to both the root and ubuntu user accounts
config:
  user.vendor-data: |
    users:
      - name: root
        ssh-import-id: gh:githubusername
        shell: /bin/bash
      - name: ubuntu
        ssh-import-id: gh:githubusername
        shell: /bin/bash
description: Default LXD profile

--snip---

```

## Sources

https://ubuntu.com/blog/howto-automatically-import-your-public-ssh-keys-into-lxd-instances

https://docs.ansible.com/ansible/latest/collections/community/general/lxd_container_module.html

