# forge

Convention-Driven Instance Autonomy for EC2

Description
-----------

Forge is designed to facilitate autonomous server configuration. At first boot, a server should execute the bootstrap code, which will in turn:
* Install the tools required for the rest of the code, using pip.
* Determine the purpose of the server, using a handful of AWS APIs.
* Download any playbooks that are applicable to the server, install their dependent roles, and execute them.

Dependencies
------------
Forge will fulfill its own dependencies, if ```pip``` is available. If it is not, the following python packages must be available.
* [ansible](https://github.com/ansible/ansible/)
* [awscli](https://aws.amazon.com/cli/)
* [boto](https://boto.readthedocs.org/)

Requirements
------------
* An S3 bucket to store roles in.
* An IAM Role to apply to autonomous servers, with a [User Policy](https://github.com/colstrom/forge/blob/master/examples/policy.json) granting access to the above bucket.
* (optional) One or more Ansible Roles in the bucket.

Self-Discovery via Conventions
------------------------------
Forge will attempt to figure out what needs to happen on its own. To do this, it relies on conventions enforced by the tools it was built to work alongside: ```meta/infrastructure``` and ```superluminal```. If you are not using these tools, Forge should work without hassle as long as you follow similar conventions.

An instance should have resource tags. Among these, we should expect to find:
* ```Project```: The project this instance belongs to.
* ```Role```: The purpose of this specific instance, within that project.
* ```ForgeBucket```: The name of the S3 bucket Forge should pull from.
* ```ForgeRegion```: Which region is ForgeBucket found in?

If sufficient resource tags are not present, Forge will make reasonable guesses. It assumes security groups are named like ```your-project-name-role```, and infers 'implicit tags' from this. Additional data can be provided via environment variables. If the instance has two security groups named ```['your-project-name-application', 'your-project-name-managed']```

* Project will be ```'your-project-name'```
* Role will be ```['application', 'managed']``` and both will be configured.
* ForgeBucket can be provided as ```FORGE_BUCKET``` in the environment.
* ForgeRegion can be provided as ```FORGE_REGION``` in the environment.

Resource tags are considered explicit statements of intent, and discovery stops there. Everything else is a fallback.

How to Use (Hardcore Mode)
--------------------------
If you're cool with allowing arbitrary code from the internet to run with root privileges with no human oversight, you can do this:

```curl https://raw.githubusercontent.com/telusdigital/forge/master/bootstrap.py | sudo python```

How to Use (Recommended)
------------------------
If you'd prefer a more sane approach, upload ```bootstrap.py``` to somewhere you control.

```curl https://YOUR_URL_HERE/bootstrap.py | sudo python```

## Setting up your environment for playbook dev

### Creating the playbook from scratch
* Create a new repository
* Populate it with the [Playbook Skeleton](https://github.com/telusdigital/playbook-skeleton) scripts
* Make sure any vault files have your master vault password in the forge bucket

### Testing changes to roles
* Make all your changes locally
* Get them to a next server (test branches, rsync, etc...) in the `/etc/ansible/roles/username.rolename` path
* Edit `/tmp/playbook-*.yaml` on the server to use your test role instead of telusdigital's
* Use `ansible-playbook` to rerun the playbook manually

### Syncing playbooks to forge
```pip install s3cmd```

```s3cmd --configure```

```s3cmd sync playbook-foo/ s3://telusdigital-forge/foo/```

### Rerun forge without reprovisioning
* SSH into the server
* Run `sudo reforge`

### Checking that everything ran properly
```cat /var/log/cloud-init-output.log```

### Troubleshooting
Q: fatal: [localhost] => One or more undefined variables: 'domain' is undefined

A: Add domain: teluswebteam.com to your infra playbook until we get it sorted out how this can be done better globally.

License
-------
[MIT](https://tldrlegal.com/license/mit-license)

Contributors
------------
* [Chris Olstrom](https://colstrom.github.io/) | [e-mail](mailto:chris@olstrom.com) | [Twitter](https://twitter.com/ChrisOlstrom)
