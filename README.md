# dnsify
Simple REST-based API for Bind DNS servers. It currently supports PUT, POST, and DELETE operations to add, update, and remove A records (forward lookups) and PTR records (reverse lookups) from an existing Bind DNS setup.

## Using the DNSify RESTful API
The API currently supports PUT, POST, and DELETE methods to add, update, and delete entries from DNS. On successful PUT a 201 is returned, on successful POST or DELETE a 200 is returned. Duplicate entries will not be created and will result in a 500 error returned.

<b>PUT</b>
A PUT request is used to create a new record in DNS. DNSify will create entries in both the forward and reverse zone files on successful PUT. A 201 is returned on success, otherwise a 500 error is returned if the entry already exists or if there was another problem.

Note: You must substitute in the IP address of the Bind DNS server for "localhost" in the examples below.
<pre>
curl -i -X PUT -H 'Content-Type: application/json' -H 'X-Api-Key: secret' -d '{ "hostname": "examplehost.xlabs.avaya.com", "ip": "10.130.124.24" }' http://localhost:4567/dns
</pre>

<b>POST</b>
A POST request is used to update/change an existing DNS entry (can change either IP or hostname for an existing entry). DNSify will attempt to update entries in both the forward and reverse zone files on successful POST. On successful POST a 200 is returned, otherwise a 500 error is returned if the entry does not exist or if there was another problem.
<pre>
curl -i -X POST -H 'Content-Type: application/json' -H 'X-Api-Key: secret' -d '{ "hostname": "examplehost.xlabs.avaya.com", "ip": "10.130.124.23" }' http://localhost:4567/dns
</pre>

<b>DELETE</b>
A DELETE request is used to delete an existing DNS entry. DNSify will remove entries in both the forward and reverse zone files on successful DELETE. On successful DELETE a 200 is returned, otherwise a 500 error is returned if the entry to delete to does not exist or if there was another problem.
<pre>
curl -i -X DELETE -H 'Content-Type: application/json' -H 'X-Api-Key: secret' -d '{ "hostname": "examplehost.xlabs.avaya.com", "ip": "10.130.124.23" }' http://localhost:4567/dns
</pre>

## Installation Instructions
Tested on Ubuntu 14.x LTS. You may run your own DNS-as-a-Service by running the code on your own Bind DNS server setup. This API can be run locally on the master bind server itself (supports master/slaves as well) or remotely on another server. You must first install ruby and use gem to install the sinatra and json gems.
<pre>
ruby dns.rb
</pre>
Example Bind configuration files have been provided to help facilitate getting started using this API with your own Bind DNS server.

## TODO
<li>Create tokenized authentication method</li>
<li>Add support for CNAME records</li>
