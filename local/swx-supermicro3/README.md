# swx-supermicro3

This is the swx-u-ub-supermicro3 server 

This consists of:

A dual-12core hyperthreaded 32G supermicro chassis with 4 internal 1.86G drives hardware RAID10 running Ubuntu 18.04

This is wired to a SAS 45 drive JBOD array, with 9 of those drives as a ZFS pool.

## Notes on how this host was added to swx-devops

1. Copied README.md here from another project. Edited it to reflect this new environment.

2. Setup ssh key trust on host for swxadmin user.

3. Setup /etc/sudoers with NOPASSWD: for the %admin group

6. Run docker-machine with the generic driver:

For the supermicro servers, with the ZFS volume driver:

    docker-machine create -d generic --generic-ip-address 172.109.152.119 --generic-ssh-port 63022 --generic-engine-port 63376 --generic-ssh-key ${devops}/secrets/ssh/sofwerx --generic-ssh-user swxadmin --engine-storage-driver overlay2 swx-u-ub-supermicro3

If you run into any problems doing this, you may safely remove this and try again:

    docker-machine rm -y swx-u-ub-supermicro3

We are running docker's `/var/lib/docker` persistence from `/dev/sda3` carved out of the 4 drive hardware RAID array.

7. This docker-machine host as a dm:

    swx dm import swx-u-ub-supermicro3

8. Create a `.dm` file for the default dm host to auto-switch when you cd to the directory:

    echo "swx-u-ub-supermicro3" > .dm

9. Setup environment variables for traefik:

    swx environment create swx-supermicro3
    swx environment set DNS_DOMAIN supermicro3.opswerx.org

10. Setup the dm2environment association between the dm host to the environment it is part of, to allow auto-switching to the environment when you cd to the directory:

    trousseau set dm2environment:swx-u-ub-supermicro3 swx-supermicro3

11. Setup environment variable for `docker-compose` to know which `.yml` file to use for this environment, and the ARCH of this environment:

    swx environment set COMPOSE_FILE swx-supermicro3.yml
    swx environment set ARCH x86_64

12. Add the `.trousseau` file to the git repo and commit and push it as a new change:

    git add ../../.trousseau
    git commit -m 'Updating secrets'
    git push

13. Add DNS records in Route53 to point `*.supermicro3.devwerx.org` to the cluster public IP
- Clone the `terraform/` directory from another project
- Edit the `Makefile`, `README.md`, and `tf.sh` to reflect this new environment
- Gut the `variables.tf` and `vpc.tf` to reflect only what is required for the AWS Route53 record resources
- Run `./tf.sh` to setup the terraform S3 bucket for this environment
- Run `terraform plan` and make sure only 2 new Route53 resources are being created.
- Run `terraform apply` to make the changes

14. Add traefik service

- Copy the `traefik:` service from another environment's `.yml` file into the `swx-pandora.yml` file (it is very generic).
- Add the `docker-traefik` submodule:

    git submodule add https://github.com/sofwerx/docker-traefik.git
    cd docker-traefik
    git remote set-url origin git@github.com:sofwerx/docker-traefik.git
    cd ..

- Deploy traefik with `docker-compose`:

    docker-compose up -d traefik

