This will be the area for any WAZUH related project/lab setups. This is still in the early stages of concept.

##Current Working Idea##

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

OWASP Juiceshop v20? I think? I'll find out when I do the docker pull

##Current Spot
I have Wazuh up, and the docker host with juiceshop running, but I'm trying to figure out container application log collection