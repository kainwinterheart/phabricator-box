# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.vm.define "master" do |config|

	mah_ip = "10.8.1.5"

	config.ssh.username = "vagrant"
	config.ssh.password = "vagrant"
        config.vm.box = "ubuntu-precise"
        config.vm.hostname = "phabricator"
        config.vm.box_url = "http://127.0.0.1/~gennadiy/ubuntu-precise.box"
	config.vm.network "private_network", ip: mah_ip
	config.vm.provision :chef_solo do |chef|
		
		chef.cookbooks_path = "cookbooks"

		chef.node_name = config.vm.hostname

		chef.add_recipe "apt"
		chef.add_recipe "hostsfile-attrs"
		chef.add_recipe "package-attrs"
		chef.add_recipe "mysql::server"

		chef.json = {

			"package-attrs" => {
			
				"install" => [
				
					{
						"name" => "php5"
					}
				]
			},
			"hostsfile-attrs" => [
			
				{
					"ip" => mah_ip,
					"host" => config.vm.hostname
				}
			]
		}

	end

	config.vm.provision :shell, inline: "sudo sh -c 'mkdir -p /opt/phabricator && cd /opt/phabricator ; if [ -d phabricator ]; then exit 0; fi; if [ -e ./install_ubuntu.sh ]; then unlink ./install_ubuntu.sh; fi ; curl http://www.phabricator.com/rsrc/install/install_ubuntu.sh -s -o install_ubuntu.sh && chmod 755 install_ubuntu.sh ; yes | ./install_ubuntu.sh'"
	config.vm.provision :shell, inline: "sudo sh -c 'cd /opt/phabricator/phabricator && if [ ! -e .storage_k ]; then ./bin/config set mysql.pass ilikerandompasswords ; ./bin/storage upgrade --force ; fi; touch .storage_k'"

	config.vm.provision :chef_solo do |chef|
		
		chef.cookbooks_path = "cookbooks"

		chef.node_name = config.vm.hostname

		chef.add_recipe "package-attrs"
		chef.add_recipe "nginx-simple"

		chef.json = {

			"package-attrs" => {
			
				"remove" => [
				
					{
						"name" => "apache2"
					},
					{
						"name" => "apache2.2-bin"
					},
					{
						"name" => "apache2-utils"
					}
				],
				"install" => [
				
					{
						"name" => "php5-fpm"
					}
				]
			},
			"nginx-simple" => {

				"server" => [
				
					{
						"name" => config.vm.hostname,
						"port" => 80,
						"locations" => [
						
							{
								"name" => "/",
								"attrs" => [

									[ "index", "index.php" ],
									[ "rewrite", "^/(.*)$", "/index.php?__path__=/$1", "last" ]
								]
							},
							{
								"name" => "/index.php",
								"attrs" => [
								
									[ "fastcgi_pass", "localhost:9000" ],
									[ "fastcgi_index", "index.php" ],
									[ "fastcgi_param", "REDIRECT_STATUS", "200" ],
									[ "fastcgi_param", "SCRIPT_FILENAME", "$document_root$fastcgi_script_name" ],
									[ "fastcgi_param", "QUERY_STRING", "$query_string" ],
									[ "fastcgi_param", "REQUEST_METHOD", "$request_method" ],
									[ "fastcgi_param", "CONTENT_TYPE", "$content_type" ],
									[ "fastcgi_param", "CONTENT_LENGTH", "$content_length" ],
									[ "fastcgi_param", "SCRIPT_NAME", "$fastcgi_script_name" ],
									[ "fastcgi_param", "GATEWAY_INTERFACE", "CGI/1.1" ],
									[ "fastcgi_param", "SERVER_SOFTWARE", "nginx/$nginx_version" ],
									[ "fastcgi_param", "REMOTE_ADDR", "$remote_addr" ]
								]
							}
						],
						"attrs" => [
						
							[ "root", "/opt/phabricator/phabricator/webroot" ]
						]
					}
				]
			}
		}
	end
    end

end
