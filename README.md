## What is this?

This is an ansible playbook for deploying instances of polis. I hope to make it
flexible enough to accomodate various environments. This should be treated as
experimental, or "alpha" software at this time. This utilized docker images
and containers for everthing. It is really just a nifty way of orchestating
docker stuff on your server.

### Why ansible and not docker-compose?

Good question. I am not totally sure if this is the right choice for most
situations. I have built up a collection of ansible playbooks over the years
of hosting various web services on bare metal web servers that I maintain.
I have come to rely on ansible for a great many tasks that are outside the
scope of docker-compose. Ansible is a much bigger tool than docker-compose and
can be extremey useful in managing everything on a server, including users,
services, backups, migrations, software requirements and more. If you are able
to get away with the simplicity of docker-compose in your environment then you
should probably stick with that and avoid the overhead of ansible.

### How do you get HTTPS working?

I use a dockerized nginx proxy and letsencrypt companion, per
https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion. And of
course I have an ansible playbook for managing those. This is the reason that
VIRTUAL_HOST and LETSENCRYPT_HOST are specified on the polis server container.
This might be overkill, but it watches your docker containers for anything with
publicly exposed ports and automatically establishes an https connection from
the outside world, with letsencrypt certification. If you are hosting multiple
web services on the same machine, then this is the way to go. There's nothing
stopping you from hosting a swarm of polises (a "skulk of poleis"?), with
various domains or subdomains, from a single server. I plan to.

### Is this ready to use?

NO! The upstream polisServer project is under active and rapid development. One
blocker is the degree to which the https://pol.is domain is hard-coded in the
source code. In its current state, this project will not fully function due
to some of these issues. Progress is being made every day and I hope to remove
this warning soon. You can get involved or monitor this progress at
https://github.com/pol-is/polisServer.

## Setup and Configuration

### prerequisites

1. ensure that your target server has python installed. I think python3 is
  required by ansible these days.
2. ensure that your target server has git installed on it and can access
  the git repository with polis, or a polis fork.
3. ensure that you have ssh access to your target server
4. ensure that your server has docker installed, the docker daemon running, and
  that your user is a member of the 'docker' group
5. ensure that your domain dns is pointing to your server's IP address
6. ensure you have ansible installed on your development machine (personal
   laptop or whatever)

    _note: I do have an ansible playbook that does most of these things on my
   server, but it is specific to Arch linux, and makes other assumptions. If I
   can make it more general I might include that in future versions of this
   project._

   _See https://github.com/aliencyborg/ansible-server_

### clone this repo

7. clone this repo on your development machine (personal laptop or whatever)

   `git clone git@github.com:ballPointPenguin/polis-playbook.git`

   `cd polis-playbook`

### configuration files

8. copy the example files:

   `cp .env.example .env`

   `cp inventory.example inventory`

   `cp vars.example.yml vars.yml`

9. edit the .env file:
    - Change DOMAIN_OVERRIDE to your domain name
    - Change PRIMARY_POLIS_URL to your domain name
    - Generate a random password for ENCRYPTION_PASSWORD_00001
    - You can probably leave everything else alone. Unfortunately this file
      is tightly coupled to the inner workings of the polis source, and is
      likely to change as things consolidate and streamline. You can change
      things like the database name and password, but you will need to make
      sure that the branch or fork of polis that you clone is compatible with
      those changes. I recommend changing as little as possible here.
10. edit the inventory file:
    - Change ansible_host to your target server IP address or domain name
    - Change ansible_private_key_file to the location of your ssh key.
    - Change ansible_user if you prefer non-root on your server. You may need
      to include some privilege escalation in your playbook or in ansible.cfg.
      Consult the ansible documentation for more information:
      - https://docs.ansible.com/ansible/latest/network/getting_started/first_inventory.html
      - https://docs.ansible.com/ansible/latest/user_guide/become.html
11. edit vars.yml:
    - change the hosts to your domain, or domains, which can include sub-
      domains and www, e.g. "mypolis.net,www.mypolis.net"
    - change the admin_email to an email address you don't mind being
      associated with your letsencrypt certificates, if using.
    - change org and branch to match the github repo you are using for polis
      source code. by default this will use github.com/pol-is/polisServer
    - optionally change the workdir that you want to use on your target server
    - optionally change the buildtag. This currently has no substantial effect
      because the playbook is building all images from source and not pushing
      or pulling to/from a docker image repository. At some point it might be
      beneficial to have this pull from e.g. docker hub, but for now my focus
      has been building fresh images based on code changes in my repository.

## Running

OK is everything configured? I hope so. This should now work:

`ansible-playbook proxy-playbook.yml`

`ansible-playbook polis-playbook.yml`

## Troubleshooting

Issues, PRs, suggestions are all welcome. I want to get this into a state that
accomodates a variety of real world use cases.

for additional debugging try

`ansible-playbook -v polis-playbook.yml`

or

`ansible-playbook -vvv polis-playbook.yml`

It can also be helpful to look at the logs from the docker containers themselves.

For example:

`$ ssh myserver`

`# docker logs polis-server`

## Further Reading
- https://roamresearch.com/#/app/polis-methods/page/1GR4r4LX8
- https://github.com/pol-is/polisServer
- https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion/wiki
- https://docs.ansible.com/ansible/latest/user_guide/playbooks.html
- https://docs.ansible.com/ansible/latest/modules/docker_image_module.html
- https://docs.ansible.com/ansible/latest/modules/docker_container_module.html
