# dnsify
Simple REST-based API for Bind DNS servers. It currently supports PUT, POST, and DELETE operations to add, update, and remove entries from a Bind DNS setup.

## Instructions
Tested on Ubuntu 14.x LTS. This API can be run locally on the master bind server itself (supports master/slaves as well) or remotely on another server. You must first install ruby and use gem to install the sinatra and json gems.
<pre>
ruby dns.rb
</pre>
Example Bind configuration files have been provided to help facilitate getting started using this API with a Bind DNS server.
