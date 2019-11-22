# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'json'
require 'yaml'

VAGRANTFILE_API_VERSION ||= "2"
confDir = $confDir ||= File.expand_path(File.dirname(__FILE__))

homesteadYamlPath = confDir + "/Homestead.yaml"
homesteadJsonPath = confDir + "/Homestead.json"
afterScriptPath = confDir + "/after.sh"
customizationScriptPath = confDir + "/user-customizations.sh"
aliasesPath = confDir + "/aliases"

require File.expand_path(File.dirname(__FILE__) + '/scripts/homestead.rb')

Vagrant.require_version '>= 2.2.4'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    if File.exist? aliasesPath then
        config.vm.provision "file", source: aliasesPath, destination: "/tmp/bash_aliases"
        config.vm.provision "shell" do |s|
            s.inline = "awk '{ sub(\"\r$\", \"\"); print }' /tmp/bash_aliases > /home/vagrant/.bash_aliases && chown vagrant:vagrant /home/vagrant/.bash_aliases"
        end
    end

    if File.exist? homesteadYamlPath then
        settings = YAML::load(File.read(homesteadYamlPath))
    elsif File.exist? homesteadJsonPath then
        settings = JSON::parse(File.read(homesteadJsonPath))
    else
        abort "Homestead settings file not found in #{confDir}"
    end

    Homestead.configure(config, settings)

    if File.exist? afterScriptPath then
        config.vm.provision "shell", path: afterScriptPath, privileged: false, keep_color: true
    end

    config.vm.define "php" do |php|
        php.vm.box = "laravel/homestead"
        php.vm.synced_folder "~/Works/Projects", "/var/www", owner: "www-data", group: "www-data"
        php.vm.synced_folder "~/Works/apache-sites-available", "/etc/apache2/sites-available", owner: "www-data", group: "www-data"
        php.vm.synced_folder "~/Works/nginx-sites-available", "/etc/nginx/sites-available", owner: "root", group: "root"
        php.vm.synced_folder "/Users/hieupv/OneDrive/Nighfury/Dev/Bin", "/usr/local/sbin", owner: "root", group: "root"
        php.vm.synced_folder "/Users/hieupv/OneDrive/Nighfury/Dev/ssl", "/etc/certificate"
        php.vm.hostname = "php.dev"
        # php.ssh.insert_key = false
        php.vm.provider "virtualbox" do |vb|
            vb.name = "php"
        end
        php.vm.network "public_network", ip: "192.168.1.199"
        php.vm.network "private_network", ip: "192.168.10.20"
    end
end
