# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'fileutils'
Vagrant.require_version '>= 1.7.0'

VAGRANT_DIR = '/vagrant'
SECRETS_DIR = '.'

CLOUD_CONFIG_PATH = File.join(File.dirname(__FILE__), 'user-data')

COREOS_LOCAL_VERSION = '>=877.1.0'

GOCD_HOST_LOCAL = '192.168.33.66'
GOCD_PORTS_LOCAL = { 8153 => 8153, 80 => 8080 }

GO_SERVER_IMAGE = 'go-server'
GO_AGENT_IMAGE = 'go-agent'
DOCKER_REGISTRY_IMAGE = 'docker-registry'
GO_AGENTS = (1..3).map {|i| GO_AGENT_IMAGE + i.to_s}

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
    cfg.vm.synced_folder './docker/go_server/godata/', '/srv/gocd/go-server', type: 'rsync'
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
      v.memory = 8192
      v.cpus = 1
      v.check_guest_additions = false
      v.functional_vboxsf     = false
   end
 end

  def import_agent_image(config)
    config.vm.provision :shell, inline: 'docker load --input /vagrant/docker/go.agent.selenium.ready.tar || true'
  end

  def import_server_image(config)
      config.vm.provision :shell, inline: 'docker load --input /vagrant/docker/go.server.tar || true'
    end

  def setup_provisioner_for_removing_existing_containers_and_cleanup_images(config)
    config.vm.provision :shell, inline: 'docker kill $(docker ps -q) || true'
    config.vm.provision :shell, inline: 'docker rm $(docker ps -a -q) || true'
  end

  def deploy_go_server_and_agent_containers(config)
    config.vm.synced_folder '.', '/vagrant', disabled: true
    config.vm.synced_folder './docker/', '/vagrant/docker', type: 'rsync'
    config.vm.synced_folder './docker/.secrets', '/srv/gocd/secrets', type: 'rsync'
    host_docker_secrets = "#{SECRETS_DIR}/docker/.secrets"
    config.vm.provision :file, source: "#{host_docker_secrets}/htpasswd", destination: "#{VAGRANT_DIR}/docker/go_server/.secrets/htpasswd"
    setup_provisioner_for_removing_existing_containers_and_cleanup_images config
    import_agent_image config
    import_server_image config
    config.vm.provision :docker do |d|
      d.build_image "#{VAGRANT_DIR}/docker/go_server/",
        args: " -t #{GO_SERVER_IMAGE} -t #{GO_SERVER_IMAGE}:latest"

      d.run GO_SERVER_IMAGE, image: "#{GO_SERVER_IMAGE}:latest", args: '--privileged -i -t -p 8153:8153 -p 8154:8154 -v /srv/gocd/go-server/config:/godata/config -v /srv/gocd/go-server/plugins:/godata/plugins -v /var/run/docker.sock:/var/run/docker.sock -v /srv/gocd/secrets:/srv/gocd/secrets'
      d.build_image "#{VAGRANT_DIR}/docker/go_agent/", args: "-t agent:latest"
      # GO_AGENTS.each do |agent|
      #   go_agent_args = "--privileged -ti --link #{GO_SERVER_IMAGE}:#{GO_SERVER_IMAGE} -e AGENT_AUTO_REGISTER_KEY=123456789abcdefgh987654321 -e AGENT_AUTO_REGISTER_HOSTNAME=#{agent} -e GO_SERVER_URL=https://#{GO_SERVER_IMAGE}:8154/go -v /var/run/docker.sock:/var/run/docker.sock -v /srv/gocd/go-agents/#{agent}/pipelines:/godata/pipelines -v /srv/gocd/go-agents/#{agent}/log:/godata/log -v /srv/gocd/secrets:/home/go/secrets"
      #   d.run agent, image: "agent:latest", args: go_agent_args
      # end

      # Run image registry
      registry_args = "-d -p 5000:5000 --restart=always --net host --restart=always"
      d.run DOCKER_REGISTRY_IMAGE, image: "registry:2",args: registry_args
    end
  end
  def deploy_cloud_config(override)
    if File.exist?(CLOUD_CONFIG_PATH)
      override.vm.provision :file, source: CLOUD_CONFIG_PATH, destination: '/tmp/vagrantfile-user-data'
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
