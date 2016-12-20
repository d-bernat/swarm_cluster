$prepare_swarm_manager_script = <<SCRIPT
service docker stop;
rm -rf /etc/docker/key.json
echo 'DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --dns 172.17.0.1 --dns 10.0.2.3 --dns 8.8.8.8 --dns-search service.consul"' | tee -a /etc/default/docker;
service docker start;
SCRIPT

$prepare_swarm_node1_script = <<SCRIPT
service docker stop;
rm -rf /etc/docker/key.json;
echo 'DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --dns 10.0.2.3"' | tee -a /etc/default/docker;
service docker start;
SCRIPT

$prepare_swarm_node2_script = <<SCRIPT
service docker stop;
rm -rf /etc/docker/key.json;
echo 'DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --dns 10.0.2.3"' | tee -a /etc/default/docker;
service docker start;
SCRIPT


Vagrant.configure(2) do |config|
  config.vm.define "swarm_manager" do |config|
    config.vm.box = "deviantony/ubuntu-14.04-docker"
    config.vm.hostname = "swarm-manager"
    config.vm.network "private_network", ip: "66.55.44.10"
    config.vm.provision "shell", inline: $prepare_swarm_manager_script
    config.vm.synced_folder ".", "/vagrant", disabled: false
  end

  config.vm.define "swarm_node1" do |config|
    config.vm.box = "deviantony/ubuntu-14.04-docker"
    config.vm.hostname = "swarm-node1"
    config.vm.network "private_network", ip: "66.55.44.11"
    config.vm.provision "shell", inline: $prepare_swarm_node1_script
    config.vm.synced_folder ".", "/vagrant", disabled: false
  end

  config.vm.define "swarm_node2" do |config|
    config.vm.box = "deviantony/ubuntu-14.04-docker"
    config.vm.hostname = "swarm-node2"
    config.vm.network "private_network", ip: "66.55.44.12"
    config.vm.provision "shell", inline: $prepare_swarm_node2_script
    config.vm.synced_folder ".", "/vagrant", disabled: false
  end


#  config.vm.define "swarm_client" do |config|
#    config.vm.box = "deviantony/ubuntu-14.04-docker"
#    config.vm.hostname = "swarm-client"
#    config.vm.network "private_network", ip: "66.55.44.13"
#    config.vm.synced_folder ".", "/vagrant", disabled: false
#  end

end
