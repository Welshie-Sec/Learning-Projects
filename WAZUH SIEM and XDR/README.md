This will be the area for any WAZUH related project/lab setups. This is still in the early stages of concept.

##Current Working Idea

Okay, so current working idea is to do this.
1. Use the Wazuh OVA, cut down on some of the initial setup.
2. Use OWASP Juice Shop, through a docker container. Will need to setup a basic linux docker machine.
3. Install the Wazuh agent on the linux machine.
4. Test Wazuh functionality, make sure what it's seeing works.
5. Pull up a Kali and start throwing things at it. Think I'll primarily use burpsuite.

Not sure yet what to record but speaking may cause issues elsewhere so I'm not sure the proper way to convey information.

##Versions of things

Wazuh 4.14.5

Ubuntu 24.04

(Need a target VM, OWASP Juice Shop needs more time to get web access logs pulled into Wazuh)

This is the intended layout I'm looking at working with.

<img width="957" height="464" alt="SimpleWazuhSetup" src="https://github.com/user-attachments/assets/cc5bc05d-f174-4f30-809b-68370803bc8e" />

##Current Spot
I have Wazuh up, and the docker host with juiceshop running, but I'm trying to figure out container application log collection.

Hmmm, so Im getting normal access logs from a test nginx, but Juice Shop doesn't like outputting any form of access log. I may have to consider another option for testing. 
I tried another docker image, with the same affect so I need to figure out what to do next with that. But hey I at least know Wazuh is up and running, and I did some copy and pasting of decoders and rulesets, so for Wazuh at least it's been a solid start I think. I can definately work with this at least and it shouldn't take too long to get a target going that plays nice for demo purposes.

###Day 1 Completion
So things completed for day 1 6/16/2026
1. Got a basic Wazuh virtual machine going.
2. Setup a basic Docker host
3. Installed Wazuh agent on Docker host
4. Modified configuration files on agent and manager to collect some docker logs.
5. Troubleshooted some potential example docker images to use for examples.
6. Applied decoder sets for docker logs that are formatted to syslog setup.
7. Applied ruleset for alerting to "GET" request from docker container derived logs.

<img width="868" height="674" alt="wazuhLogin" src="https://github.com/user-attachments/assets/9692a022-b2e2-4091-b83d-f0e1fd80c912" />

<img width="1902" height="938" alt="wazuhdecoder" src="https://github.com/user-attachments/assets/81b619c4-2ee6-499a-8834-423f40a1d1bd" />

<img width="1913" height="903" alt="wazuhrules" src="https://github.com/user-attachments/assets/ad4d13be-aef4-4db4-9841-eb60d8394db7" />

<img width="1911" height="1031" alt="ThreatHunting_Dashboard" src="https://github.com/user-attachments/assets/49a6892a-8bf5-434a-88a0-821f562bfdfb" />




