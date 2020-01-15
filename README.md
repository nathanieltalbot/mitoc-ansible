[![Build Status](https://travis-ci.com/DavidCain/mitoc-ansible.svg?branch=master)](https://travis-ci.com/DavidCain/mitoc-ansible)

# MITOC Ansible

This is a collection of Ansible playbooks used in the deployment of the [MITOC
Trips web site][mitoc-trips], [MITOC membership & waiver
processing][mitoc-member], and related infrastructure.
The project allows local development on a setup closely mirroring
that of production as well as a streamlined way to deploy changes.

## Environments
There are playbooks supporting three separate environments:

### Local development (`vagrant.yml`)
For local development, all services run within a virtual machine. Vagrant is
used to automate the creation and modification of this virtual machine. To
create a new virtual machine and provision it with Ansible, simply run:

```bash
vagrant up
```

After that, you will have a fully functional web server accessible at
https://mitoc-trips.local

### Development (`development.yml`)
This playbook allows running all infrastructure on AWS. By default, it runs a
Postgres server on the same instance as the webserver, but could easily be
configured to use RDS instead.

Just like a local development machine, the EC2 instance can easily be created
using Vagrant.

1. Store [AWS credentials][aws-credentials] in `.aws/credentials` or directly in the `Vagrantfile`.
2. Create a [keypair][aws-keypairs] or import an existing one.
3. Install and configure `vagrant-aws`:

    ```bash
    vagrant plugin install vagrant-aws
    vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
    ```

4. Launch a new EC2 instance:

    ```bash
    vagrant up --provider=aws
    ```

### Production (`production.yml`)
The production playbook contains various secrets used in production, including (but
not limited to):

- The Django `SECRET_KEY`
- Full SSL certificate for mitoc-trips.mit.edu
- Usernames, hostnames, and passwords for various services, including:
    - RabbitMQ
    - Postgres
    - SES

It also uses some more time-intensive plays that wouldn't be necessary in development
(for example - generating a [strong Diffie-Hellman group][weakdh]).

For obvious reasons, secrets within this file are encrypted using Ansible
vault's [`encrypt_string`][encrypt-string].

In the public version of this repository, I have used `git filter-branch` to
redact encrypted secrets (if we make the encrypted secrets open source, one could
theoretically brute-force the Ansible-vault password).

This repository originally used `ansible-vault` to encrypt entire files, but
later transitioned to using [`encrypt_string`][encrypt-string]. The following
multi-line regular expression substitution can redact both types of secret
storage in Git history:

```
# When !vault is present, it indicates output from `encrypt_string`.
# When !vault is absent, we're matching (and replacing) an entire file encrypted with Ansible-vault
regex='(!vault \|\n)?'

# The `$ANSIBLE_VAULT` string indicates encrypted contents from `ansible-vault`.
# The capture group includes the version and the algorithm used for encryption
regex+='\s*\$ANSIBLE_VAULT;(.*)'

# All lines following `$ANSIBLE_VAULT` are output from Python's `hexify()`
regex+='(\n\s*[0-9a-f]+)*'

# Finally, we capture the final newline
regex+='\n'

# Any encrypted content is replaced, inline, with 'REDACTED'
perl -0777 -i -pe "s/$regex/REDACTED\n/g" $1;
```

Production (an EC2 instance running Ubuntu server) is deployed directly with Ansible:

```bash
ansible-playbook -i hosts production.yml -u ubuntu --private-key=<path_to_iam_user_key>
```


## History
The repository is a modified fork of Johnathan Calazan's
[ansible-django-stack][ansible-django-stack].

## Included services
- Nginx
- Gunicorn
- PostgreSQL
- Supervisor
- Virtualenv
- Celery
- RabbitMQ

[aws-credentials]: http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#config-settings-and-precedence
[aws-keypairs]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
[ansible-django-stack]: https://github.com/jcalazan/ansible-django-stack
[encrypt-string]: https://docs.ansible.com/ansible/latest/user_guide/vault.html?highlight=encrypt_string#use-encrypt-string-to-create-encrypted-variables-to-embed-in-yaml
[weakdh]: https://weakdh.org/sysadmin.html

[mitoc-member]: https://github.com/DavidCain/mitoc-member
[mitoc-trips]: https://github.com/DavidCain/mitoc-trips
