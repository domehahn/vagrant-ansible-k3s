VAGRANTFILE_API_VERSION = "2"
NUM_NODES = (ENV['NUM_NODES'] || 0).to_i
NETWORK_PREFIX = ENV['NETWORK_PREFIX'] || "10.135.135"

def provider(override, docker)
    override.vm.box = nil
    override.ssh.insert_key = true
    docker.has_ssh = true
    docker.privileged = true
    docker.build_dir = "."
    #docker.image = "ubuntu"
end

def private_network(node, node_sum)
    #node.network "private_network", ip: "#{NETWORK_PREFIX}.#{100+node_sum}",
    #    auto_config: false
    node.network "private_network", type: "dhcp"
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    if Vagrant.has_plugin?("vagrant-timezone")
        config.timezone.value = :host
    end

    if NUM_NODES==0 or NUM_NODES>0
        config.vm.define "k8s-master" do |master|
            private_network(master.vm, 0)
            master.vm.network "forwarded_port", guest: 6443, host: 6443,
                auto_correct: true
            master.vm.network "forwarded_port", guest: 6444, host: 6444,
                auto_correct: true
            master.vm.provider "docker" do |docker, override|
                provider(override, docker)
                docker.name = "k8s-master"
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
                    docker.name = "k8s-node-#{i}"
                end
            end
        end
    end

    config.vm.provision "ansible_local" do |ansible|
        ansible.become = true
        ansible.playbook = "provisioning/site.yml"
        ansible.limit = "all"
        ansible.groups = { 
                            "master" => [ 
                                "k8s-master"
                            ],
                            "node" => [ 
                                "k8s-node-1",
                                "k8s-node-2"
                            ],
                            "k3s_cluster" => [ 
                                "k8s-master",
                                "k8s-node-1",
                                "k8s-node-2"
                            ]
        }
        ansible.host_vars = {
                            "k8s-master" => {
                                "master_host" => "10.135.135.100", 
                                "master_token" => "K103ba45cbc2e93bbc7c1b9a9eb104cb982d074cc330930fbb655502f3557e9f519::server:3b6c99796a27ec48b528f311496e9636"
                            }
        }
        ansible.galaxy_role_file = "provisioning/roles/requirements.yml"
        ansible.galaxy_roles_path = "/etc/ansible/roles"
        ansible.galaxy_command = "sudo ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
        ansible.verbose = true
    end
end                     