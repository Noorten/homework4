MACHINES = {
  :otuslinux => {
    :box_name => "centos/7",
    :ip_addr => '192.168.56.51',
    :disks => {
      :sata1 => {
        :dfile => './sata1.vdi',
        :size => 250,
        :port => 1
      },
      :sata2 => {
        :dfile => './sata2.vdi',
        :size => 250, # Megabytes
        :port => 2
      },
      :sata3 => {
        :dfile => './sata3.vdi',
        :size => 250,
        :port => 3
      },
      :sata4 => {
        :dfile => './sata4.vdi',
        :size => 250, # Megabytes
        :port => 4
      },
      :sata5 => {
        :dfile => './sata5.vdi',
        :size => 250, # Megabytes
        :port => 5
      }
    }
  }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      box.vm.network "private_network", ip: boxconfig[:ip_addr]

      box.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "1024"]
        needsController = false

        boxconfig[:disks].each do |dname, dconf|
          unless File.exist?(dconf[:dfile])
            vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
            needsController = true
          end
        end

        if needsController == true
          vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata"]
          boxconfig[:disks].each do |dname, dconf|
            vb.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
          end
        end
      end

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        yum install -y mdadm smartmontools hdparm gdisk
        mdadm --zero-superblock --force /dev/sd[b-f]
        mdadm --create --verbose /dev/md0 --level=0 --raid-devices=5 /dev/sd[b-f]
        mkfs.ext4 /dev/md0
        mkdir -p /mnt/raid0
        mount /dev/md0 /mnt/raid0
        mdadm --detail --scan >> /etc/mdadm.conf
        dracut -H -f /boot/initramfs-$(uname -r).img $(uname -r)
        echo '/dev/md0 /mnt/raid0 ext4 defaults 0 0' >> /etc/fstab
      SHELL
    end
  end
end
