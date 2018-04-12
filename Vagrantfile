# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'fileutils'
Vagrant.require_version '>= 1.7.0'

VAGRANT_DIR = '/vagrant'
SECRETS_DIR = '.'

CLOUD_CONFIG_PATH = File.join(File.dirname(__FILE__), 'user-data')

COREOS_LOCAL_VERSION = '>=877.1.0'

GOCD_HOST_LOCAL = '192.168.33.66'
GOCD_PORTS_LOCAL = { 80 => 8153 }

GO_SERVER_IMAGE = 'go17_server'
GO_AGENT_IMAGE = 'go17_agent'
DOCKER_REGISTRY_IMAGE = 'docker_registry'
GO_AGENTS_IMAGES = (1..1).map {|i| GO_AGENT_IMAGE + i.to_s}

Vagrant.configure('2') do |config|
  config.ssh.insert_key = false

  # plugin conflict
  config.vbguest.auto_update = false if Vagrant.has_plugin?('vagrant-vbguest')

  config.vm.define 'gocd-local' do |cfg|
    your_private_key_which_has_to_be_authorized_in_server_manually = '~/.vagrant.d/insecure_private_key'
    cfg.vm.box = 'tknerr/managed-server-dummy'
    cfg.vm.provider :managed do |managed, override|
      managed.server = GOCD_HOST_LOCAL
      override.ssh.username = 'core'
      override.ssh.private_key_path = your_private_key_which_has_to_be_authorized_in_server_manually
      deploy_cloud_config(override)
    end
    deploy_go_server_and_agent_containers(cfg)
  end

  config.vm.define 'local-mcloud-impostor' do |cfg|
    # Provisioning for this box is done by the managed `gocd-local` box
    cfg.vm.provider :virtualbox do |v, override|
      override.vm.network :private_network, ip: GOCD_HOST_LOCAL
      GOCD_PORTS_LOCAL.each do |guest, host|
         override.vm.network 'forwarded_port', guest: guest, host: host
      end
      override.vm.hostname = 'localhost'
      override.vm.box = 'coreos-alpha'
      override.vm.box_version = COREOS_LOCAL_VERSION
      override.vm.box_url = 'http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json'
      v.memory = 3072
      v.check_guest_additions = false
      v.functional_vboxsf     = false
   end
 end

  def setup_provisioner_for_removing_existing_containers_and_cleanup_images(config)
    config.vm.provision :shell, inline: 'docker rm -f `docker ps -aq` 2>/dev/null || true'
    config.vm.provision :shell, inline: 'docker rmi `docker images -f dangling=true -q` 2>/dev/null || true'
  end

  def deploy_go_server_and_agent_containers(config)
    config.vm.synced_folder '.', '/vagrant', disabled: true
    config.vm.synced_folder './docker/', '/vagrant/docker', type: 'rsync'
    host_docker_secrets = "#{SECRETS_DIR}/docker/.secrets"
    config.vm.provision :file, source: "#{host_docker_secrets}/htpasswd", destination: "#{VAGRANT_DIR}/docker/go_server/.secrets/htpasswd"
    setup_provisioner_for_removing_existing_containers_and_cleanup_images config

    config.vm.provision :docker do |d|
      d.build_image "#{VAGRANT_DIR}/docker/go_server/",
        args: "-t #{GO_SERVER_IMAGE} -t #{GO_SERVER_IMAGE}:latest"

      d.run GO_SERVER_IMAGE, image: "#{GO_SERVER_IMAGE}:latest", args: '-i -t --net host -p 80:8153 -p8154:8154 -v /srv/gocd/go-server:/godata'

      GO_AGENTS_IMAGES.each do |agent|
        d.build_image "#{VAGRANT_DIR}/docker/go_agent/", args: "-t #{agent} -t #{agent}:latest --build-arg agent=#{agent}"
        go_agent_args = "--privileged -ti --net host -e AGENT_AUTO_REGISTER_KEY=123456789abcdef -e AGENT_AUTO_REGISTER_HOSTNAME=#{agent} -v /var/run/docker.sock:/var/run/docker.sock -v /srv/gocd/go-agents/#{agent}:/godata -v /srv/gocd/secrets:/home/go/secrets"
        d.run agent, image: "#{agent}:latest", args: go_agent_args
      end

      # Run image registry
      registry_args = "-d -p 5000:5000 --net host --restart=always"
      d.run DOCKER_REGISTRY_IMAGE, image: "registry:2",args: registry_args
    end
  end

  def deploy_cloud_config(override)
    if File.exist?(CLOUD_CONFIG_PATH)
      override.vm.provision :file, source: CLOUD_CONFIG_PATH, destination: '/tmp/vagrantfile-user-data'
      GO_AGENTS_IMAGES.each do |agent|
        script = <<-eos
        mkdir -p /srv/gocd/go-agents/#{agent}/pipelines
        mkdir -p /srv/gocd/go-agents/#{agent}/logs
        chown -R 1000:1000 /srv/gocd/go-agents/#{agent}
        eos
      end

      script = <<-eos
      chown -R 1000:1000 /srv/gocd/go-server
      chown -R 1000:1000 /srv/gocd/secrets
      mkdir -p /var/lib/coreos-vagrant/ && mv -f /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/
      chmod a+rw /var/run/docker.sock
      eos
      override.vm.provision :shell, inline: script, privileged: true
    end
  end
end
