VAGRANTFILE_API_VERSION = "2"
NUM_NODES = (ENV['NUM_NODES'] || 0).to_i
NETWORK_PREFIX = ENV['NETWORK_PREFIX'] || "10.135.135"

def provider(override, docker)
    override.vm.box = nil
    override.ssh.insert_key = true
    docker.has_ssh = true
    docker.privileged = true
    docker.build_dir = "."
end

def private_network(node, node_sum)
    node.network :private_network, ip: "#{NETWORK_PREFIX}.#{100+node_sum}"
end

ansible_groups = { 
    "master" => [ 
        "k8s-master"
    ],
    "node" => [ 
        "k8s-node-1",
        "k8s-node-2"
    ],
    "k3s_cluster:children" => [ 
        "master",
        "node"
    ]
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    if Vagrant.has_plugin?("vagrant-timezone")
        config.timezone.value = :host
    end

    if NUM_NODES==0 or NUM_NODES>0
        config.vm.define "k8s-master" do |master|
            private_network(master.vm, 0)
            master.vm.network "forwarded_port", guest: 6443, host: 6443
            master.vm.network "forwarded_port", guest: 6444, host: 6444
            master.vm.provider "docker" do |docker, override|
                provider(override, docker)
                docker.ports = ["6443:6443", "6444:6444"]
            end
        end
    end
    if NUM_NODES>0
        (1..NUM_NODES).each do |i|
            config.vm.define "k8s-node-#{i}" do |node|
                private_network(node.vm, i)
                node.vm.provider "docker" do |docker, override|
                    provider(override, docker)
                end
            end
        end
    end

    config.vm.provision "ansible_local" do |ansible|
        ansible.become = true
        ansible.playbook = "provisioning/site.yml"
        ansible.config_file = "ansible.cfg"
        ansible.limit = "all"
        #ansible.groups =  ansible_groups
        ansible.galaxy_role_file = "provisioning/roles/requirements.yml"
        ansible.galaxy_roles_path = "/etc/ansible/roles"
        ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
        ansible.verbose = true
    end
end                     