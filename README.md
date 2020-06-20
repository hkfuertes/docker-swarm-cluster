Ansible playbooks to deploy Docker Swarm on a Raspberry Pi Cluster
-----------------------------------------------------------------------------

- Requires six Raspberry Pi devices (v3 was used in the demo, v2 also works fine)
- Requires six micro-SD cards (8GB)
- Requires Ansible (2.2 was used in the demo)
- Requires [HypriotOS](https://blog.hypriot.com/downloads/)
- Requires HypriotOS installed on the micro-SD cards
- Requires hostnames configured on the micro-SD cards matching the names in the inventory file


These playbooks have been demonstrated at the "DockerGrunn Meetup #7 - Winter is coming" event on 18 January 2017. The slide deck is available at [Speaker Deck](https://speakerdeck.com/tisgoud/dockergrunn-meetup-number-7-docker-swarm-on-a-raspberry-pi-cluster).

### Set up SSH

If you use homebrew then follow the steps below otherwise follow the instructions on [How to set up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)

Install ssh-copy-id with brew:

		apt install ssh-copy-id

Repeat for each node:

		ssh-copy-id -i ~/.ssh/id_rsa.pub pirate@[name]

Use the name(s) of the swarmManager and swarmNodes from the inventory file

### Initial Swarm Setup

In the demo the following inventory file is used:

		# Inventory file for Raspberry Pi cluster

		[swarmManagers]
		blacknode.local

		[swarmNodes]
		greennode.local
		rednode.local
		pinknode.local
		yellownode.local
		whitenode.local

		[swarm:children]
		swarmManagers
		swarmNodes

		[swarm:vars]
		ansible_ssh_user=pirate

To deploy the swarm cluster you execute the following command:

		ansible-playbook -i inventory swarmSetup.yml

The first time this playbook is executed it takes a couple of minutes. A docker image is downloaded and this takes some time.

The following steps will be executed:

- The visualization container is started
- The swarm is initialized
- On the machine running Ansible the visualization page url is displayed
- The nodes are added to the cluster

### Teardown sequence

To clean up the environment we execute the setup steps in the reverse order.

Teardown the cluster:

		ansible-playbook -i inventory swarmTeardown.yml

Shutdown the Raspberry Pi stack:

		ansible-playbook -i inventory shutdown.yml

### Pushing and Pulling images to registry
There is a registry also deployed as part of the swarm, to push and pull images from/to it just use:

```
docker push <managarNode_ip>:5000/<image_name>:<image_tag>
docker pull <managarNode_ip>:5000/<image_name>:<image_tag>
```