# dnsify
Simple REST-based API for Bind DNS servers. It currently supports PUT, POST, and DELETE operations to add, update, and remove A records (forward lookups) and PTR records (reverse lookups) from an existing Bind DNS setup. Tested with Bind 9.

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

## Installation Instructions
You can create your own DNS-as-a-Service by running this code on your existing Bind DNS setup. This API can be run locally on the master Bind server itself (supports master/slave setups) or remotely from another server. You must first install ruby and use gem to install the sinatra and json gems. Tested on Ubuntu 14.x LTS and Bind 9. 

<pre>
ruby dns.rb -o [ip_address]
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
As mentioned, DNSify utilizes nsupdate to modify the zone files. Nsupdate is a dynamic DNS update utility, so if there is a manual change that must be made to a zone under DNSify's control then you will need to first freeze the zones before making the manua edits to the zone file, then you must thaw them in order to put them back into service. Use the following commands when making manual edits:

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
<li>Add support for CNAME records</li>
