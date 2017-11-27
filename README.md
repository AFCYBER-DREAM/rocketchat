Rocket.Chat
===========

The purpose of this role is to prep a system to run Rocket.Chat in a pair of docker containers and ensure that the needed services are running. Post install configuration will need to be manually done.

Requirements
------------

Currently a copy of the containers will need to be copied to the server. This can be via a thumb drive or from a web server using wget or curl. Once a docker registry is in place there should be fewer steps to finalizing the server.

This role is to be paired with the `lvm_partition` and `docker-setup` roles. Make sure that you have both available.

You should have 3 drives on your system (assuming sdx here)

* /dev/sda = OS
* /dev/sdb = unpartitioned. Will be set up to be used for /var/lib/docker. About 20G should be sufficient.
* /dev/sdc = unpartitioned. Will be set up to be used for /var/lib/docker/volumes. About 100G should be sufficient.

Role Variables
--------------

None

Dependencies
------------

This role should be paired with the `lvm_partition` and `docker-setup` roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    ---
    - import_playbook: infrastructure.yml

    - name: Set up rocketchat server
      hosts: rocketchat
      become: true
      gather_facts: true

      roles:
        - lvm_partition
        - docker
        - rocketchat

Initial Post Provisioning Operations
------------------------------------

1. Load the tgz versions of the two containers into docker locally.
```bash
docker load -i /root/mongodb.tgz
docker load -i /root/rocketchat.tgz
```
2. Create a volume for the mongodb data.
```bash
docker volume create --name mongodb
```
3. Start the mongodb container using the created volume.
```bash
docker run -d -p 27017:27017 -v mongodb:/data/db --name mongodb --restart always mongodb:latest
```
4. Start the rocketchat container linking it to the mongodb container.
```bash
docker run --name rocketchat -p 80:3000 -v rocketchat:/app/uploads --env ROOT_URL=http://localhost --link mongodb:db -d --restart always rocket.chat
```
5. Use the following settings to set up AD authentication

```
Domain Base: DC=EXAMPLE,DC=COM
Custom Domain Search: {"filter": "(&(objectCategory=person)(objectclass=user)(sAMAccountName=#{username}))", "scope": "sub", "userDN": "<user>@example.com", "password": "<user_password>"}
Domain Search User: CN=USER CN,OU=OU WHERE USER LIVES,DC=EXAMPLE,DC=COM
Domain Search Password: <user_password>
user group filter: False 
Username Field: sAMAccountName 
Sync Data: True 
Sync User Avatar: False 
User Data Map Field: {"cn":"name", "mail":"email"} 
Default Domain: EXAMPLE.COM
```

License
-------

MIT

Author Information
------------------

The Development Range Engineering, Architecture, and Modernization (DREAM) Team.
