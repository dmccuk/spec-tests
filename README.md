# spec-tests
ServerSpec installation walk-through
ServerSpec install
Install the following packages:

  * For the purposes of this test, completed the following steps on your **Jenkins server as the ubuntu user**

````
# cd /opt
# sudo git clone https://github.com/dmccuk/spec-tests.git
# cd spec-tests
# sudo apt-get install ruby bundler  
````

Check this file
````
# cat Gemfile 
source 'https://rubygems.org'
gem 'serverspec'
gem 'rake', '~>12.0.0'
gem 'net-ssh', '~>3.0.2'
````

Run the following commands:
````
# sudo bundle
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and installing your bundle as root will break this application
for all non-root users on this machine.
Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/..
Installing rake 12.0.0
Installing diff-lcs 1.3
Installing multi_json 1.13.1
Installing net-ssh 3.0.2
Using net-telnet 0.1.1
Installing rspec-support 3.8.0
Installing sfl 2.3
Using bundler 1.11.2
Installing net-scp 2.0.0
Installing rspec-core 3.8.0
Installing rspec-expectations 3.8.2
Installing rspec-mocks 3.8.0
Installing specinfra 2.77.0
Installing rspec-its 1.3.0
Installing rspec 3.8.0
Installing serverspec 2.41.3
Bundle complete! 3 Gemfile dependencies, 16 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.

# sudo gem install serverspec
Fetching: serverspec-2.41.5.gem (100%)
Successfully installed serverspec-2.41.5
Parsing documentation for serverspec-2.41.5
Installing ri documentation for serverspec-2.41.5
Done installing documentation for serverspec after 0 seconds
1 gem installed

# sudo serverspec-init 

Select OS type:
  1) UN*X
  2) Windows
Select number: 1
Select a backend type:
  1) SSH
  2) Exec (local)
Select number: 1
Vagrant instance y/n: n
Input target host name: web01.example.com
 + spec/
 + spec/www.example.jp/
 + spec/www.example.jp/sample_spec.rb
 + spec/spec_helper.rb
 + Rakefile
 + .rspec

# cat Rakefile 
require 'rake'
require 'rspec/core/rake_task'
hosts = %w(
  <server1>
  <server2>
)
task :spec => 'spec:all'
namespace :spec do
  task :all => hosts.map {|h| 'spec:' + h.split('.')[0] }
  hosts.each do |host|
    short_name = host.split('.')[0]
    role       = short_name.match(/[^0-9]+/)[0]
    desc "Run serverspec to #{host}"
    RSpec::Core::RakeTask.new(short_name) do |t|
      ENV['TARGET_HOST'] = host
      t.pattern = "spec/{base,#{role}}/*_spec.rb"
      t.rspec_opts = "--format documentation --format html --out /opt/spec-tests/reports/#{host}.html"
    end
  end
end
````

Now check the pre-defined tests:
````
# cat /opt/spec-tests/spec/base/base_spec.rb 
require 'spec_helper'
describe package('nginx') do
  it { should be_installed }
end
describe service('nginx') do
  it { should be_enabled }
  it { should be_running }
end
describe port(80) do
  it { should be_listening }
end
describe port(22) do
  it { should be_listening }
end
````

Run these commands
````
# sudo mkdir /opt/spec-tests/reports 
# sudo chmod 777 reports
````

<details>
 <summary>Only do the following if you don't already have an SSH-KEY PAIR setup. <NOT if you're doing the Devops Tools course!></summary>
  <p>
    
Setup a private/public key pair:
````
# cd /home/vagrant 
````
Just keep hitting enter until it finishes for the command below:
````
# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
fe:2a:77:61:21:dc:fd:8e:5e:6b:88:5b:2f:07:70:d8 vagrant@puppet
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|       . . +     |
|        o = E    |
|        S. + .   |
|       .  o . .  |
|        .. o.=.  |
|      . ..o.+o+. |
|       o.ooo.+o  |
+-----------------+
# cat .ssh/id_rsa.pub 
...Copy...your...public...key..output...
````
Now login to web01. You probably already have it open:
````
vagrant@web01:~$ vi .ssh/authorized_keys 

 * Press SHIFT+G 
 * then - o 
Now cut and paste your Key from the puppet master in here.
 * Press - Esc 
 * Then - SHIFT+: 
 * Now press - wq!  ENTER
````
Test your key works.

Back on the puppet master run the following command:
````
# cd 
#  ssh <server> uptime 
(the first time you will get a message prompt - Answer Yes)
 06:55:36 up 1 day, 14:01,  0 users,  load average: 0.00, 0.04, 0.07
````
If you get this we can move on to running the tests:

</p></details>

Update the Rakefile with the servers you want to run against. This will be the FQDN of your AWS servers:
````
hosts = %w(
  ec2-18-197-99-47.eu-central-1.compute.amazonaws.com
  ec2-3-125-42-34.eu-central-1.compute.amazonaws.com
)
````

Before we run our tests against our webservers, let change the ownershop of the serverspec over to the jenkins user. 

As the Ubuntu user:
````
$ cd /opt
$ sudo chown -R jenkins. spec-tests/
$ sudo su - jenkins
$ cd /opt/spec-tests
````

Now you are ready to run the tests against your webservers
````
$ rake spec
/usr/bin/ruby2.3 -I/var/lib/gems/2.3.0/gems/rspec-support-3.8.0/lib:/var/lib/gems/2.3.0/gems/rspec-core-3.8.0/lib /var/lib/gems/2.3.0/gems/rspec-core-3.8.0/exe/rspec --pattern spec/\{base,ec\}/\*_spec.rb

Package "nginx"
  should be installed

Service "nginx"
  should be enabled
  should be running

Port "80"
  should be listening

Port "22"
  should be listening

Finished in 2.55 seconds (files took 1.49 seconds to load)
5 examples, 0 failures
````

Congratulations! You have successfully tested the actual state of your infrastructure.

Check https://serverspec.org/resource_types.html for more tests.

## Reports:

Copy one of the reports generated to the /vagrant folder on the devops-vm. You can access this from your laptop and open the webpage to view the results.

If you are using the vagrant VM, copy the report to your PC using the following command:
````
vagrant@devops-box:/opt/spec-tests$ cp reports/ec2-3-8-126-230.eu-west-2.compute.amazonaws.com.html /vagrant/
````


