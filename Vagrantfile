# -*- mode: ruby -*-
# vi: set ft=ruby :

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
#
# Place this Vagrantfile in your src folder and run:
#
#     vagrant up
#
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

# Vagrantfile API/syntax version.
VAGRANTFILE_API_VERSION = "2"

Vagrant.require_version ">= 1.5.0"

GO_VERSION = "1.8rc3"

# See http://dl.golang.org/dl/
GO_ARCHIVES = {
  "linux" => "go#{GO_VERSION}.linux-amd64.tar.gz",
  "bsd" => "go#{GO_VERSION}.freebsd-amd64.tar.gz",
  "source" => "go#{GO_VERSION}.src.tar.gz"
}

INSTALL = {
  "linux" => "apt-get update -qq; apt-get install -qq -y git",
  "bsd" => "pkg install -y wget git",
  "solaris" => "pfexec pkg install build-essential"
}

# location of the Vagrantfile
def src_path
  File.dirname(__FILE__)
end

# shell script to bootstrap Go
def bootstrap(box)
  install = INSTALL[box]
  archive = GO_ARCHIVES[box]
  source = GO_ARCHIVES["source"]
  bootstrap = "go1.4-bootstrap-20161024.tar.gz"

  downloadURL="https://storage.googleapis.com/golang"

  profile = <<-PROFILE
    export GOPATH=$HOME
    export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
    export CDPATH=.:$GOPATH/src/github.com:$GOPATH/src/golang.org
  PROFILE

  <<-SCRIPT
  #{install}

  if [ -n "#{archive}" ]; then
    if ! [ -f /home/vagrant/#{archive} ]; then
      response=$(wget -nv #{downloadURL}/#{archive})
    fi
    tar -C /usr/local -xzf #{archive}
  else
    mkdir -p /usr/local/go/bootstrap
    if ! [ -f #{bootstrap} ]; then
      wget -nv #{downloadURL}/#{bootstrap}
    fi
    tar -C /usr/local/go/bootstrap -xzf #{bootstrap}
    bash -c "cd /usr/local/go/bootstrap/go/src && ./make.bash"

    if ! [ -f #{source} ]; then
      wget -nv #{downloadURL}/#{source}
    fi
    tar -C /usr/local -xzf #{source}
    bash -c "cd /usr/local/go/src && GOROOT_BOOTSTRAP=/usr/local/go/bootstrap/go ./make.bash"
  fi

  home=~vagrant
  echo '#{profile}' >> ${home}/.profile

  echo "\nRun: vagrant ssh #{box} -c 'cd project/path; go test ./...'"
  SCRIPT
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "linux" do |linux|
    linux.vm.box = "ubuntu/trusty64"
    linux.vm.synced_folder src_path, "/home/vagrant/src"
    linux.vm.provision :shell, :inline => bootstrap("linux")
  end

  # Using NFS for synced/shared folders:
  # * which is unsupported on Windows hosts (though maybe with freeNFS)
  # * and requires a private host-only network
  # * and will prompt for the administrator password of the host
  # * but is said to be faster than VirtualBox shared folders
  config.vm.define "bsd" do |bsd|
    bsd.vm.box = "bento/freebsd-10.3"
    bsd.vm.synced_folder ".", "/vagrant", :disabled => true
    bsd.vm.synced_folder src_path, "/home/vagrant/src", :nfs => true
    bsd.vm.network :private_network, :ip => '10.1.10.5'
    bsd.vm.provision :shell, :inline => bootstrap("bsd")
    bsd.ssh.shell = "sh" # for provisioning
  end

  config.vm.define "solaris" do |solaris|
    solaris.vm.box = "openindiana/hipster"
    solaris.vm.synced_folder src_path, "/export/home/vagrant/src"
    solaris.vm.provision :shell, :inline => bootstrap("solaris")
  end

end
