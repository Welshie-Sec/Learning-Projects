This will be the area for any WAZUH related project/lab setups. This is still in the early stages of concept.

---

## Current Working Idea

Okay, so current working idea is to do this.

1. Use the Wazuh OVA, cut down on some of the initial setup.
2. Use OWASP Juice Shop, through a docker container. Will need to setup a basic linux docker machine.
3. Install the Wazuh agent on the linux machine.
4. Test Wazuh functionality, make sure what it's seeing works.
5. Pull up a Kali and start throwing things at it. Think I'll primarily use burpsuite.

Not sure yet what to record but speaking may cause issues elsewhere so I'm not sure the proper way to convey information.

---

## Versions of things

* **Wazuh:** 4.14.5
* **Ubuntu:** 24.04

*(Need a target VM, OWASP Juice Shop needs more time to get web access logs pulled into Wazuh)*

This is the intended layout I'm looking at working with.

---

## Current Spot

I have Wazuh up, and the docker host with juiceshop running, but I'm trying to figure out container application log collection.

Hmmm, so Im getting normal access logs from a test nginx, but Juice Shop doesn't like outputting any form of access log. I may have to consider another option for testing.

I tried another docker image, with the same affect so I need to figure out what to do next with that. But hey I at least know Wazuh is up and running, and I did some copy and pasting of decoders and rulesets, so for Wazuh at least it's been a solid start I think. I can definately work with this at least and it shouldn't take too long to get a target going that plays nice for demo purposes.

### Day 1 Completion

So things completed for day 1 6/16/2026

1. Got a basic Wazuh virtual machine going.
2. Setup a basic Docker host
3. Installed Wazuh agent on Docker host
4. Modified configuration files on agent and manager to collect some docker logs.
5. Troubleshooted some potential example docker images to use for examples.
6. Applied decoder sets for docker logs that are formatted to syslog setup.
7. Applied ruleset for alerting to "GET" request from docker container derived logs.

#### Gallery

<img width="868" height="674" alt="wazuhLogin" src="https://github.com/user-attachments/assets/9692a022-b2e2-4091-b83d-f0e1fd80c912" />

<img width="1902" height="938" alt="wazuhdecoder" src="https://github.com/user-attachments/assets/81b619c4-2ee6-499a-8834-423f40a1d1bd" />

<img width="1913" height="903" alt="wazuhrules" src="https://github.com/user-attachments/assets/ad4d13be-aef4-4db4-9841-eb60d8394db7" />

<img width="1911" height="1031" alt="ThreatHunting_Dashboard" src="https://github.com/user-attachments/assets/49a6892a-8bf5-434a-88a0-821f562bfdfb" />


### Day 2 Completion

A little shorter of an entry. 6/27/2026

1. Started using docker-compose and bringing up basic nginx container that feeds out it's access and error logs and using it as a reverse proxy for JuiceShop as a test.
2. I can see most events coming from the access logs, but it seems like Wazuh isn't doing anything with 200 code log entries. Like in the threat hunting it shows 400 and 500 base errors.
3. Since I'm not completely familiar with what Wazuh would alert from the get go I wanted to run some basic things that I feel it could alert on, but with mixed results.
4. One issue I ran into when doing something basic like using dirbuster is that dirbuster was totally freaking out about getting responses back, so I need to test if it's because of the reverse proxy setup or if dirbuster is not happy.
5. Burpsuite intruder worked fine with the expected outputs on non-existant directories. I'll probably grab a request file from burp and throw it through wfuzz later for a loud directory enumeration.

<img width="1901" height="874" alt="ThreatHunting_Showing400Codes" src="https://github.com/user-attachments/assets/8844e6d2-682b-40b3-b13f-5cd34cd3b4e2" />
*Showing pickups of nginx proxy logs*

#### Basic docker-compose.yml

```yaml
version: '3'
services:
  juiceshop:
    image: bkimminich/juice-shop
    container_name: juiceshop
    expose:
      - "3000"

  nginx:
    image: nginx:latest
    container_name: nginx-proxy
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx-logs:/var/log/nginx
    depends_on:
      - juiceshop

```

#### nginx.conf

```nginx
events {}

http {
    server {
        listen 80;

        location / {
            proxy_pass http://juiceshop:3000;
            proxy_set_header HOST $http_host;
            proxy_set_header X-REAL-IP $remote_addr;
        }
    }
}
```
