# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  required_plugins = %w(vagrant-timezone vagrant-proxyconf vagrant-docker-compose)
  plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
  if not plugins_to_install.empty?
    puts "Installing plugins: #{plugins_to_install.join(' ')}"
    if system "vagrant plugin install #{plugins_to_install.join(' ')}"
      exec "vagrant #{ARGV.join(' ')}"
    else
      abort "Installation of one or more plugins has failed. Aborting."
    end
  end

  config.vm.box = "ubuntu/xenial64"
  config.vm.box_check_update = false
  config.vbguest.auto_update = false
  
  config.timezone.value = :host
  
  if ENV["http_proxy"]
    puts "http_proxy: " + ENV["http_proxy"]
    config.proxy.http = ENV["http_proxy"]
  end
  if ENV["https_proxy"]
    puts "https_proxy: " + ENV["https_proxy"]
    config.proxy.https = ENV["https_proxy"]
  end
  if ENV["no_proxy"]
    puts "no_proxy: " + ENV["no_proxy"]
    config.proxy.no_proxy = ENV["no_proxy"]
  end

  config.vm.define "grafana-docker" do |node|
    node.vm.hostname = "grafana-docker"
    node.vm.network "private_network", ip: "10.10.10.10"
    node.vm.provider "virtualbox" do |v|
      v.name = "grafana-docker"
      v.cpus = "1"
      v.memory = "256"
    end

    node.vm.provision "docker" do |d|
      d.post_install_provision "shell", path: "scripts/set-docker-mirror.sh"
    end
    node.vm.provision "shell", path: "scripts/install-docker-compose.sh"
    node.vm.provision "docker_compose",
      compose_version: "1.16.1",
      yml: "/vagrant/grafana-docker/docker-compose.yaml",
      rebuild: false,
      run: "always"

    node.vm.provision "shell", path: "scripts/bootstrap.sh"
  end
end