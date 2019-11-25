Ansible play to test out logstash and ILM.

Requires Vagrant and Ansible. 

### cd to vagrant directory and run 
```
vagrant up

cd ..
```
### Run the following command to kick off the ansible play
```
ansible-playbook -i inventory.yml ansible-playbook.yml
```
### log into kibana
```
http://192.168.50.35:5601
```
### Create the policy via the dev_tools
```
PUT _ilm/policy/test-logs
{
    "policy" : {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "50gb",
              "max_docs" : "20"
            }
          }
        }
      }
    }
  }
```
### Check that the policy was created successfully
```GET _ilm/policy/test-logs```

### Start filebeat
```
vagrant ssh ls-node1-rpm

sudo bin/filebeat -e -c /etc/filebeat/filebeat.yml -d "publish"
```
### start logstash
```
vagrant ssh ls-node1-rpm
sudo /usr/share/logstash/bin/logstash -f /usr/share/logstash/pipeline2.conf --config.reload.automatic --path.settings /etc/logstash/
```
### Run the following commands via the Dev Tools in kibana 
### Check the index template to see if it was created correctly
```
GET _template/test-threat-logs
```
### Check the ILM status. Did it roll over correctly?
```
GET test-threat-logs*/_ilm/explain
```
if the log files fill the index too quickly due to indices.lifecycle.poll_interval being set to low, just run sudo yum update to update the os packages and that should create some more log files.