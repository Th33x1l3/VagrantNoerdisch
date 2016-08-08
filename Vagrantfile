# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_VERSION = "20160805.1"
BOX_CHECKSUM = "51211988680be77e72805ab913ba36324b9f57ffaa77349109c42a8752efe827"

HOST_DB = "192.168.50.51"
HOST_WEB = "192.168.50.50"

CPUS = `sysctl -n hw.physicalcpu`.to_i
MEMORY = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 8

if MEMORY.to_i < 1024
    MEMORY = 1024
end

Vagrant.configure("2") do |config|
    config.vm.box = "noerdisch/ubuntu-16.04/#{BOX_VERSION}"
    config.vm.box_url = "https://cdn.noerdisch.net/vagrant/images/ubuntu/16.04/#{BOX_VERSION}/ubuntu-16.04-server-amd64_virtualbox.box"
    config.vm.box_check_update = false
    config.vm.box_download_checksum = "#{BOX_CHECKSUM}"
    config.vm.box_download_checksum_type = "sha256"

    config.vm.define "phoenix-web", primary: true do |box|
        box.vm.network "private_network", ip: HOST_WEB
        box.vm.synced_folder "_vHosts", "/var/www", type: "nfs"
        box.vm.synced_folder "_transfer", "/opt/transfer", type: "nfs"
        box.vm.hostname = "phoenix-web"
        box.vm.provider "virtualbox" do |vb|
            vb.name      = "Noerdisch Development Stack (web)"
            vb.gui       = false
            vb.cpus      = CPUS.to_i
            vb.memory    = MEMORY.to_i

            vb.customize [ "modifyvm", :id, "--natdnshostresolver1", "on" ]
            vb.customize [ "modifyvm", :id, "--natdnsproxy1",        "on" ]
            vb.customize [ "modifyvm", :id, "--nictype1",        "virtio" ]
        end

        box.vm.provision "docker" do |docker|
            docker.run "mailhog", image: "mailhog/mailhog", daemonize: true, args: "--publish 127.0.0.1:1025:1025 --publish 0.0.0.0:8025:8025 --env MH_HOSTNAME=mail.local.noerdisch.net"
        end
    end

    config.vm.define "phoenix-db" do |box|
        box.vm.network "private_network", ip: HOST_DB
        box.vm.synced_folder "_transfer", "/opt/transfer", type: "nfs"
        box.vm.hostname = "phoenix-db"
        box.vm.provider "virtualbox" do |vb|
            vb.name      = "Noerdisch Development Stack (db)"
            vb.gui       = false
            vb.cpus      = CPUS.to_i
            vb.memory    = MEMORY.to_i

            vb.customize [ "modifyvm", :id, "--natdnshostresolver1", "on" ]
            vb.customize [ "modifyvm", :id, "--natdnsproxy1",        "on" ]
            vb.customize [ "modifyvm", :id, "--nictype1",        "virtio" ]
        end
    end

    config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

    config.vm.provision :shell, :inline => "cp /vagrant/puppet/Puppetfile /etc/puppet"
    config.vm.provision :shell, :inline => "cd /etc/puppet && rm -rf /etc/puppet/modules && librarian-puppet install"
    config.vm.provision :shell, :inline => "apt-get -qy update"

    config.vm.provision "puppet" do |puppet|
        puppet.facter = {
            "db_host"            => HOST_DB,
            "web_host"           => HOST_WEB,
            "default_password"   => "jolt200mg",
            "login_user"         => `echo $(echo $USER | head -c 1 | tr [a-z] [A-Z]; echo $USER | tail -c +2) | tr -d '\n'`
        }

        puppet.manifests_path    = "puppet/manifests"
        puppet.module_path       = "puppet/modules"
        puppet.hiera_config_path = "puppet/hiera.yaml"
        puppet.manifest_file     = "noerdsite.pp"
        puppet.options           = "--environment=local --parser=future"
    end
end
