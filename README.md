# dnsify
Simple REST-based API for Bind DNS servers. It currently supports PUT, POST, and DELETE operations to add, update, and remove A records (forward lookups) and PTR records (reverse lookups) from an existing Bind DNS setup. Tested with Bind 9.x and Ruby 2.2.3.

## Using the DNSify RESTful API
The API currently supports PUT, POST, and DELETE methods to add, update, and delete entries from DNS. On successful PUT a 201 is returned, on successful POST or DELETE a 200 is returned. Duplicate entries will not be created and will result in a 500 error returned.

<b>PUT</b>
A PUT request is used to create a new record in DNS. DNSify will create entries in both the forward and reverse zone files on successful PUT. A 201 is returned on success, otherwise a 500 error is returned if the entry already exists or if there was another problem.

<i>Note: You must substitute in the IP address of the server that is running the API code for "localhost" in the examples below.</i>
<pre>
curl -i -X PUT -H 'Content-Type: application/json' -H 'X-Api-Key: secret' -d '{ "hostname": "examplehost.xlabs.avaya.com", "ip": "10.130.124.24" }' http://localhost:4567/dns
</pre>

<b>POST</b>
A POST request is used to update/change an existing DNS entry (can change either IP or hostname for an existing entry). DNSify will attempt to update entries in both the forward and reverse zone files on successful POST. On successful POST a 200 is returned, otherwise a 500 error is returned if the entry does not exist or if there was another problem.
<pre>
curl -i -X POST -H 'Content-Type: application/json' -H 'X-Api-Key: secret' -d '{ "hostname": "examplehost.xlabs.avaya.com", "ip": "10.130.124.23" }' http://localhost:4567/dns
</pre>

<b>DELETE</b>
A DELETE request is used to delete an existing DNS entry. DNSify will remove entries in both the forward and reverse zone files on successful DELETE. On successful DELETE a 200 is returned, otherwise a 500 error is returned if the entry specified to delete does not exist or if there was another problem.
<pre>
curl -i -X DELETE -H 'Content-Type: application/json' -H 'X-Api-Key: secret' -d '{ "hostname": "examplehost.xlabs.avaya.com", "ip": "10.130.124.23" }' http://localhost:4567/dns
</pre>

## Create your own DNS-as-a-Service
You can create your own DNS-as-a-Service by running this code against your existing Bind DNS setup. This API can be run locally on the master Bind server itself (supports master/slave setups) or remotely from another server. 

We are currently using this on Ubuntu 14.04.4 LTS and Ruby 2.2.3. Before you begin, you'll need to make sure you are able to access the superuser account on your Ubuntu server (e.g. you must be able to issue sudo commands).

<b>Install rbenv, Ruby and Gems</b>
First, install rbenv (which is used to install and manage the ruby installation) and the ruby dependencies:

<pre>
$ sudo apt-get update

$ sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev
</pre>

Now we can install rbenv. Run these commands (as the user that will be using Ruby, e.g. probably your normal user and not the sudo/superuser):

<pre>
$ cd
$ git clone git://github.com/sstephenson/rbenv.git .rbenv
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
$ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bash_profile
$ source ~/.bash_profile
</pre>

This installs rbenv into your home directory and sets the appropriate environment variables that will allow rbenv to use the active version of Ruby.

Now we can move on to installing Ruby. We are using ruby v.2.2.3. Again, as the user that will be using Ruby, install it with these commands:

<pre>
$ rbenv install -v 2.2.3
$ rbenv global 2.2.3
</pre>

<i>The "global" sub-command sets the default version of Ruby that all of your shells will use. If you want to install and use a different version, simply run the rbenv commands with a different version number.</i>

Verify that Ruby was installed properly:

<pre>
$ ruby -v
</pre>

You will likely not want Rubygems to generate local documentation for each gem that you install, as this process can be time consuming. To disable this:

<pre>
$ echo "gem: --no-document" > ~/.gemrc
</pre>

You will also want to install the bundler gem, it manages your application dependencies:

<pre>
$ gem install bundler
</pre>


Lastly, install the sinatra and json gems required by DNSify:

<pre>
gem install sinatra
gem install json
</pre>

<b>Bind DNS Server Installation</b>
Now we are ready to move onto the Bind DNS installation. For sake of simplicity we will assume you are running Bind on the same server as DNSify. We are currently using Bind v9.9.5.

--bind install instructions go here--

<b>Fire Up DNSify</b>
It's time to turn on DNSify so we can start consuming our API and partially free ourselves from the drudgery of manual DNS management :)

--instructions to clone DNSify??--

<pre>
$ ruby dns.rb -o [ip_address]
</pre>

Replace the IP address with the IP address of the server you are running this code on (this may or may not be the IP address of the Bind server).

You'll also need to add your rndc-key on the line shown below:

<pre>
dns_params = {
  :server => '127.0.0.1',
  :rndc_key => 'rndc-key',
  :rndc_secret => 'uHHMi8hBYlHmjHe/8f6gAg==',
  :ttl => '300'
}
</pre>

<b>Running the API on a Remote Server</b>
If you want to run the API remotely (e.g. on a different server than the DNS server) then substitute in the IP address of your Bind DNS server for the "server" parameter shown in the code snippet above.

Example Bind configuration files have been provided to help facilitate getting started using this API with your own Bind DNS server.

## Master/Slave Setup Support
DNSify utilizes the nsupdate utility to modify the Bind DNS zone files, so it works well with Bind master/slave setups since nsupdate increments the "serial" parameter in the zone files' SOA records any time a change is made.

When a slave name server contacts a master server for zone data, it first asks for the "serial" number on the data. If the slave's serial number for the zone is lower than the master server's, the slave's zone data is out of date. In this case, the slave pulls a new copy of the zone. The slave will contact the master server for zone data based on the "refresh" interval configured in the zone's SOA record. In the example below our refresh interval is set to 8 hours, meaning that the slaves will poll the master every 8 hours for zone data changes:

<pre>
xlabs.avaya.com         IN SOA  dns1.xlabs.avaya.com. root.xlabs.avaya.com. (
                                13         ; serial
                                28800      ; refresh (8 hours)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                38400      ; minimum (10 hours 40 minutes)
                                )
</pre>

## Manual Zone Changes
As mentioned, DNSify utilizes nsupdate to modify the zone files. Nsupdate is a dynamic DNS update utility, so if there is a manual change that must be made to a zone under DNSify's control then you will need to first freeze the zones before making the manual edits to the zone file, then you must thaw them in order to put them back into service. Use the following commands when making manual edits:

<pre>
rndc freeze
[make your manual edits]
rndc reload
rndc thaw
</pre>

## Daemonized
DNSify runs in the background as a linux daemon when started and uses the built-in Ruby webserver called Puma. To check if it is running, use:

<pre>
ps aux | grep "puma"
</pre>

You will also probably want to make sure DNSify comes back automatically after a server reboot. On Ubuntu, simply edit the /etc/rc.local file to something similar to this:

<pre>
# Note full path to ruby executable
/home/user/.rbenv/shims/ruby /home/user/dns.rb -o 10.130.124.18

if [ $? -eq 0 ]; then
    logger Success: DNSify has been started
    exit 0
else
    logger Error: DNSify is unable to start
    exit 1
fi
</pre>

You can test this without actually needing to reboot by using:

<pre>
service rc.local start
</pre>

## TODO
<li>Create token-based authentication method</li>
<li>Enhance error checking to support more informative return values instead of just a generic 500 all the way around</li>
<li>Add support for CNAME records</li>
