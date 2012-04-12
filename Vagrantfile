Vagrant::Config.run do |config|
  config.vm.define :master do |master|
    master.vm.box = "lucid64"
    master.vm.customize ["modifyvm", :id, "--memory", "384"]
    master.vm.host_name = "master.vagrant.icisapp.com"
    master.vm.network :hostonly, "33.33.33.12"
  end
  config.vm.define :slave do |slave|
    slave.vm.box = "lucid64"
    slave.vm.customize ["modifyvm", :id, "--memory", "384"]
    slave.vm.host_name = "slave.vagrant.icisapp.com"
    slave.vm.network :hostonly, "33.33.33.13"
  end
end