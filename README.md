# spec-tests
ServerSpec installation walk-through
ServerSpec install
Install the following packages:

# cd /opt
# git clone https://github.com/dmccuk/spec-tests.git
# sudo apt-get install ruby bundler  

Create this file:

# sudo vi Gemfile 

Add this contents:
==below this line==

source 'https://rubygems.org'
gem 'serverspec'
gem 'rake', '~>12.0.0'
gem 'net-ssh', '~>3.0.2'

==above this line==

Run the following commands:

# sudo bundle 
# sudo gem install serverspec 
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
# sudo cp Rakefile Rakefile.OLD 
# sudo vi Rakefile 

Hold down "d" to delete all the contents of the file. Then press "Esc" and "i" to insert.

Add this contents:

==below this line==

require 'rake'
require 'rspec/core/rake_task'
hosts = %w(
   web01
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
==above this line==

Now to create our tests:

# sudo mkdir /opt/spec-tests/spec/base 
# sudo vi /opt/spec-tests/spec/base/base_spec.rb 

Add this contents:


==below this line==

require 'spec_helper'
describe package('nginx') do
  it { should be_installed }
end
describe service('puppet') do
  it { should be_enabled }
  it { should be_running }
end
describe port(80) do
  it { should be_listening }
end
describe port(22) do
  it { should be_listening }
end

==above this line==

Run these commands

# cd /opt/spec-tests 
# sudo mkdir reports 
# sudo chmod 777 reports 

Setup a private/public key pair:

# cd /home/vagrant 

Just keep hitting enter until it finishes for the command below:

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

Now login to web01. You probably already have it open:

vagrant@web01:~$ vi .ssh/authorized_keys 

 * Press SHIFT+G 
 * then - o 
Now cut and paste your Key from the puppet master in here.
 * Press - Esc 
 * Then - SHIFT+: 
 * Now press - wq!  ENTER

Test your key works.

Back on the puppet master run the following command:

# cd 
#  ssh <server> uptime 
(the first time you will get a message prompt - Answer Yes)
 06:55:36 up 1 day, 14:01,  0 users,  load average: 0.00, 0.04, 0.07

# If you get this we can move on to running the tests:

# cd /opt/spec-tests 

# rake spec
/usr/bin/ruby1.9.1 -I/var/lib/gems/1.9.1/gems/rspec-support-3.6.0/lib:/var/lib/gems/1.9.1/gems/rspec-core-3.6.0/lib /var/lib/gems/1.9.1/gems/rspec-core-3.6.0/exe/rspec --pattern spec/\{base,app,users,network,ssh,\<new\ folder\?\>,web\}/\*_spec.rb --format documentation --format html --out /opt/spec-tests/reports/web01.html
Package "nginx"
  should be installed
Service "puppet"
  should be enabled
  should be running
Port "80"
  should be listening
Port "22"
  should be listening
Finished in 0.92661 seconds (files took 0.62907 seconds to load)
5 examples, 0 failures

Congratulations!
