# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Look for project settings file.
  if !(File.exists?("#{File.dirname(__FILE__)}/vars.yml"))
    raise NoSettingsException
  end

  # Load the yml files
  require 'yaml'
  settings = YAML.load_file("#{File.dirname(__FILE__)}/vars.yml")
  settings.merge!(YAML.load_file("#{File.dirname(__FILE__)}/vars.local.yml"))

  # Clone project repo to `./app` if the folder doesn't exist yet, and the setting exists.
  if !(File.directory?("#{File.dirname(__FILE__)}/app"))
    system("git clone #{settings['app_repo']} #{File.dirname(__FILE__)}/app")
  end

  # Config the VM
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = settings['server_hostname']

  config.nfs.map_uid = 1010
  config.nfs.map_gid = 1010

  # Sets IP of the guest machine and allows it to connect to the internet.
  config.vm.network :private_network, ip:  settings['vagrant_ip']
  config.vm.network :public_network, bridge: settings['vagrant_adapter']

  # Sync .ssh folder to guest machine.
  config.vm.synced_folder "#{Dir.home}/.ssh", "/home/vagrant/.ssh_host"

  # Read this user's host machine's public ssh key to pass to ansible.
  if !(File.exists?("#{Dir.home}/.ssh/id_rsa.pub"))
     raise NoSshKeyException
  end
  ssh_public_key = IO.read("#{Dir.home}/.ssh/id_rsa.pub").strip!

  # ONLY WORKS if ansible is setup on the HOST machine.
  # See https://github.com/mitchellh/vagrant/issues/2103
  # @TODO: Uncomment once vagrant supports this.
  # config.vm.provision "ansible" do |ansible|
  #   ansible.playbook = 'playbook.yml'
  # end

  # Setup ansible and run the playbook.
  # @TODO: Remove once vagrant supports ansible on guest.
  config.vm.provision "shell", path: "tasks/setup-ansible.sh"

  # Run ansible Provisioner via shell.
  config.vm.provision "shell",
    inline: "cd /vagrant; ansible-playbook -c local  -i 'default,' playbook.yml --extra-vars 'authorized_keys=\"#{ssh_public_key}\"'"

  # Extra provisioning for vagrant
  config.vm.provision "shell",
    inline: "cd /vagrant; ansible-playbook -c local  -i '#{settings['server_hostname']},' playbook.vagrant.yml"

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", settings['vagrant_memory']]
  end

end

##
# Our Exceptions
#
class NoSettingsException < Vagrant::Errors::VagrantError
  error_message('Project settings file not found. Copy settings.project.example.yml to settings.project.yml, edit to match your project, then try again.')
end

class NoSrcException < Vagrant::Errors::VagrantError
  error_message('Could not create ./src folder. Run as the owner of this folder. ')
end

class NoSshKeyException < Vagrant::Errors::VagrantError
  error_message('An ssh public key could not be found at ~/.ssh/id_rsa.pub. Please generate one and try again.')
end