# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_NAME = ENV["BOX_NAME"] || "phusion/ubuntu-14.04-amd64"
BOX_URI = ENV["BOX_URI"] || "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
BOX_MEMORY = ENV["BOX_MEMORY"] || "512"
DOKKU_DOMAIN = ENV["DOKKU_DOMAIN"] || "dokku.me"
DOKKU_IP = ENV["DOKKU_IP"] || "10.0.0.2"
PREBUILT_STACK_URL = File.exist?("#{File.dirname(__FILE__)}/stack.tgz") ? 'file:///root/dokku/stack.tgz' : nil

make_cmd = "make install"
if PREBUILT_STACK_URL
  make_cmd = "PREBUILT_STACK_URL='#{PREBUILT_STACK_URL}' #{make_cmd}"
end


Vagrant::configure("2") do |config|
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
  config.vm.box = BOX_NAME
  config.vm.box_url = BOX_URI
  config.vm.synced_folder File.dirname(__FILE__), "/root/dokku"
  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.hostname = "#{DOKKU_DOMAIN}"
  config.vm.network :private_network, ip: DOKKU_IP

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    # Ubuntu's Raring 64-bit cloud image is set to a 32-bit Ubuntu OS type by
    # default in Virtualbox and thus will not boot. Manually override that.
    vb.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
    vb.customize ["modifyvm", :id, "--memory", BOX_MEMORY]
  end

  base_cmd = "echo 'running shell ...';"

  docker_cmd = "wget -q -O - https://get.docker.io/gpg | apt-key add -;" \
      "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list;" \
      "apt-get update -qq; apt-get install -q -y --force-yes lxc-docker git software-properties-common nginx aufs-tools apache2-utils linux-image-extra-virtual;"
    # Add vagrant user to the docker group
  docker_cmd << "usermod -a -G docker vagrant;"


  dokku_cmd = "cd /root/dokku && #{make_cmd}"

  dokku_plugin_cmd = "cd /var/lib/dokku-alt/plugins ;"\
     "git clone https://github.com/musicglue/dokku-user-env-compile.git user-env-compile ;"\
     "dokku plugins-install ;"\

  shipyard_cmd = "docker run -d -P --name rethinkdb shipyard/rethinkdb ;"\
        "docker run -d -p 8080:8080 --link rethinkdb:rethinkdb shipyard/shipyard  ;"\
        "shipyard cli> shipyard add-account -u hello -p shipyard -r user ;"

  base_cmd << docker_cmd
  base_cmd << dokku_cmd
  # base_cmd << dokku_plugin_cmd
  # base_cmd << shipyard_cmd

  config.vm.provision :shell, :inline => base_cmd

  # run cat ~/.ssh/id_rsa.pub | vagrant ssh -- sudo sshcommand acl-add dokku ${USER}

end
