# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.require_version ">= 1.4"

ROOT = File.dirname(File.absolute_path(__FILE__))

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = '2'

# Default env properties which can be overridden
# Example overrides:
#   echo "ENV['PASSENGER_DOCKER_PATH'] ||= '../../phusion/passenger-docker'   " >> ~/.vagrant.d/Vagrantfile
#   echo "ENV['BASE_BOX_URL']          ||= 'd\:/dev/vm/vagrant/boxes/phusion/'" >> ~/.vagrant.d/Vagrantfile
BASE_BOX_URL          = ENV['BASE_BOX_URL']    || 'https://oss-binaries.phusionpassenger.com/vagrant/boxes/latest/'
VAGRANT_BOX_URL       = ENV['VAGRANT_BOX_URL'] || BASE_BOX_URL + 'ubuntu-14.04-amd64-vbox.box'
VMWARE_BOX_URL        = ENV['VMWARE_BOX_URL']  || BASE_BOX_URL + 'ubuntu-14.04-amd64-vmwarefusion.box'
BASEIMAGE_PATH        = ENV['BASEIMAGE_PATH' ] || '.'
PASSENGER_DOCKER_PATH = ENV['PASSENGER_PATH' ] || '../passenger-docker'
DOCKERIZER_PATH       = ENV['DOCKERIZER_PATH'] || '../dockerizer'
DOCKERHOST_MEMSIZE    = ENV['DOCKERHOST_MEMSIZE'] || '1024'
DOCKERHOST_IPADDR     = ENV['DOCKERHOST_IPADDR'] || ''
DOCKER_FWDPORT_MIN    = ENV['DOCKER_FWDPORT_MIN'] || '48000'
DOCKER_FWDPORT_MAX    = ENV['DOCKER_FWDPORT_MAX'] || '48199'
DOCKER_RUNREGISTRY    = ENV['DOCKER_RUNREGISTRY'] || 'N'
DOCKER_REGISTRY_PATH  = ENV['DOCKER_REGISTRY_PATH'] || './docker-registry'

$script = <<SCRIPT
su - vagrant -c 'echo alias d=docker >> ~/.bash_aliases'
cp /vagrant/docker_default /etc/default/docker
/vagrant/install-tools.sh
mkdir -p /home/vagrant/.ssh
cp /usr/local/share/baseimage-docker/insecure_key /home/vagrant/.ssh/id_rsa
chown -R vagrant.vagrant /home/vagrant/.ssh
chmod -R go= /home/vagrant/.ssh
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = 'phusion-open-ubuntu-14.04-amd64'
  config.vm.box_url = VAGRANT_BOX_URL
  config.ssh.forward_agent = true
  passenger_docker_path = File.absolute_path(PASSENGER_DOCKER_PATH, ROOT)
  if File.directory?(passenger_docker_path)
    config.vm.synced_folder passenger_docker_path, '/vagrant/passenger-docker'
  end
  baseimage_path = File.absolute_path(BASEIMAGE_PATH, ROOT)
  if File.directory?(baseimage_path)
    config.vm.synced_folder baseimage_path, "/vagrant/baseimage-docker"
  end
  dockerizer_path = File.absolute_path(DOCKERIZER_PATH, ROOT)
  if File.directory?(dockerizer_path)
    config.vm.synced_folder dockerizer_path, '/vagrant/dockerizer'
  end
  docker_registry_path = File.absolute_path(DOCKER_REGISTRY_PATH, ROOT)
  if File.directory?(baseimage_path)
    config.vm.synced_folder docker_registry_path, '/vagrant/docker-registry'
  end

  if DOCKERHOST_IPADDR != ''
    web.vm.network :private_network, ip: DOCKERHOST_IPADDR
  end

  config.vm.provider :virtualbox do |v|
	  v.memory = DOCKERHOST_MEMSIZE
  end

  config.vm.provider :vmware_fusion do |f, override|
    override.vm.box_url = VMWARE_BOX_URL
    f.vmx['displayName'] = 'baseimage-docker'
    f.vmx['memsize'] = DOCKERHOST_MEMSIZE
  end

  if Dir.glob("#{File.dirname(__FILE__)}/.vagrant/machines/default/*/id").empty?
    config.vm.provision :shell, :inline => $script

  end

  config.vm.provision :docker do |d|
    if DOCKER_RUNREGISTRY =~ /^[Yy]/
      d.run "registry", args: "-e STORAGE_PATH=/mnt -v /vagrant/docker-registry:/mnt -p 5000:5000"
    end
  end

  # Port range for use by docker containers
  (DOCKER_FWDPORT_MIN..DOCKER_FWDPORT_MAX).each do |port|
    config.vm.network :forwarded_port, :host => port, :guest => port
  end
end
