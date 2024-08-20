# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.define "harborMachine" do |harbor|
    config.vm.network "public_network", bridge: "enp6s0", ip: "192.168.68.120" # Substitua pelo nome da sua interface de rede e um IP válido na sua rede local
    harbor.vm.provider "virtualbox" do |vb|
        vb.memory = "4096"
        vb.cpus = "2"
    end
    harbor.vm.provision "shell", inline: <<-SHELL
      export REGISTRY=fabiopisc.devops.com # Substitua pelo seu domínio
      export IP_ADDRESS=$(curl -s ipinfo.io/ip)
      apt-get update && apt-get install -y curl
      hostname harbor-machine-PISC
      echo "127.0.0.1 harbor-machine-PISC" >> /etc/hosts
      curl -fsSL https://get.docker.com | bash
      printf '{\n    "insecure-registries" : [ "%s" ]\n}' "$REGISTRY" | sudo tee /etc/docker/daemon.json > /dev/null
      sudo groupadd docker
      sudo usermod -aG docker $USER
      sudo systemctl daemon-reload
      sudo systemctl restart docker
      curl -O -L "https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64"
      sudo mv cosign-linux-amd64 /usr/local/bin/cosign
      sudo chmod +x /usr/local/bin/cosign
      mkdir /home/ubuntu/certs && cd /home/ubuntu/certs
      openssl genrsa -out $REGISTRY.key 4096
      openssl req -x509 -new -nodes -sha512 -days 3650 \
          -subj "/C=CN/ST=Linuxtips/L=Linuxtips/O=Linuxtips/OU=Linuxtips/CN=$REGISTRY" \
          -key $REGISTRY.key \
          -out $REGISTRY.crt
      openssl req -sha512 -new \
          -subj "/C=CN/ST=Linuxtips/L=Linuxtips/O=Linuxtips/OU=Linuxtips/CN=$REGISTRY" \
          -key $REGISTRY.key \
          -out $REGISTRY.csr
      
      cat /tmp/harbor/v3.ext >> v3.ext
      cp v3.ext v3.ext.temp
      envsubst < v3.ext.temp > v3.ext
      openssl x509 -req -sha512 -days 3650 \
          -extfile v3.ext \
          -CA $REGISTRY.crt -CAkey $REGISTRY.key -CAcreateserial \
          -in $REGISTRY.csr \
          -out $REGISTRY.crt

      wget https://github.com/goharbor/harbor/releases/download/v2.10.0/harbor-offline-installer-v2.10.0.tgz
      tar -xvzf harbor-offline-installer-v2.10.0.tgz
      sudo rm harbor-offline-installer-v2.10.0.tgz
      cd harbor
      envsubst < /tmp/harbor/harbor.yml.tmpl > harbor.yml
      sudo ./install.sh --with-trivy
    SHELL
    harbor.vm.synced_folder "./harbor", "/tmp/harbor"
  end
end