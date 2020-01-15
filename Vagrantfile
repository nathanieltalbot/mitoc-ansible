# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Hack to gain access to the --tags argument
ANSIBLE_TAGS=ENV['ANSIBLE_TAGS']

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/bionic64"
  config.ssh.forward_agent = false

  config.vm.define "mitoc-trips.local", primary: true do |app|
    app.vm.hostname = "mitoc-trips"

    app.vm.network "private_network", type: "dhcp"
  end

  config.vm.provider :virtualbox do |vb, override|
    override.vm.provision "ansible" do |ansible|
      ansible.playbook = "vagrant.yml"
      ansible.host_key_checking = false
      ansible.verbose = "vv"
      ansible.tags = ANSIBLE_TAGS
    end

    override.vm.network :private_network, ip: "192.168.33.15"
    vb.customize ["modifyvm", :id, "--name", "MITOC Trips", "--memory", "1024"]
  end

  config.vm.provider :aws do |aws, override|
    override.vm.box = "dummy"

    override.vm.provision "ansible" do |ansible|
      ansible.playbook = "production.yml"
      ansible.host_key_checking = false
      ansible.verbose = "vv"
      ansible.tags = ANSIBLE_TAGS
    end

    # These can be hardcoded, in env vars, or in `~/.aws/credentials`
    #aws.access_key_id = ""
    #aws.secret_access_key = ""

    aws.ami = "ami-fce3c696"
    aws.instance_type = "t2.micro"
    aws.region = "us-east-1"
    aws.security_groups = ["webserver"]

    # Explicit defaults
    aws.tenancy = "default"

    # This is a keypair used to access your instance with `vagrant ssh`
    # AWS must have this keypair configured for your account.
    # https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
    aws.keypair_name = "vagrant"
    override.ssh.username = "ubuntu"
    override.ssh.private_key_path = "#{Dir.home}/vagrant.pem"
  end

  config.vm.provider :docker do |d, override|
    override.vm.box = nil

    d.name = "ws"
    d.build_dir = "docker"
    d.create_args = ["--publish-all", "--security-opt=seccomp:unconfined",
                     "--tmpfs=/run", "--tmpfs=/run/lock", "--tmpfs=/tmp",
                     "--volume=/sys/fs/cgroup:/sys/fs/cgroup:ro"]
    d.has_ssh = true
  end

  # For local development, uncommenting and editing the line below will enable
  # a folder in the host machine containing your local git repo to be synced to
  # the guest machine. Ensure the Ansible playbook variable "setup_git_repo" is
  # set to "no" (in env_vars/vagrant.yml) when enabling this.
  #config.vm.synced_folder "../../../my-cool-app", "/webapps/mycoolapp/my-cool-app"
end
