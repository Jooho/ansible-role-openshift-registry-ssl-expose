Ansible Role: Openshift Docker Registry Securing and Exposing Service
=========

This role help create a custom certificate for openshift integrated docker registry and apply the cert on docker registry.
Then, expose docker registry service to accept external access

Requirements
------------
None

Role Variables
--------------

| Name                             | Default value                         |        Requird       | Description                                                                 |
|----------------------------------|---------------------------------------|----------------------|-----------------------------------------------------------------------------|
| ca_dir                           | /etc/origin/master                    |         no           | Where ca cert is                                                            |
| temp_dir                         | /tmp/docker-registry-ca               |         no           | Temp directory                                                              |
| docker_registry_route_hostname   | docker-registry.cloudapps.example.com |         yes          | Docker registry Hostname                                                    |
| docker_registry_svc_ip           | ''                                    |         yes          | Docker registry service ip                                                  |
| docker_registry_svc_name         | docker-registry                       |         yes          | Docker registry service name                                                |
| use_self_signed_cert             | true                                  |         no           | Generate self signed cert                                                   |
| docker_registry_cert_file        | ' '                                   |         no           | Specify cert file to apply( must set use_self_signed_cert to false)         |
| docker_registry_key_file         | ' '                                   |         no           | Specify private key file to apply (must set use_self_signed_cert to false)  |
| replace_cert                     | no                                    |         no           | If set yes, it replaces existing cert "registry-secret"                     |
| restart_docker                   | no                                    |         no           | If set yes, it restarts docker engine on all nodes                          |


Dependencies
------------

None



Example Playbook
----------------
~~~
- name: Example Playbook
  hosts: all
  gather_facts: false

  roles:
    - { role: Jooho.openshift-registry-ssl-expose, docker_registry_route_hostname: 'docker-registry.cloudapps.example.com', docker_registry_svc_ip: '172.30.156.119', docker_registry_svc_name: 'docker-registry'}
~~~

After Job
---------

- If you set restart_docker to no, please *restart docker daemon on All nodes*


- Copy ca.crt file to the host *where access to docker registry with route*
~~~
$ sudo mkdir -p /etc/docker/certs.d/{{docker_registry_route_hostname}}"
$ sudo scp $MASTER_NODE:/etc/origin/master/ca.crt /etc/docker/certs.d/{{docker_registry_route_hostname}}"
$ sudo systemctl restart docker"
~~~

Test
------
- Login docker registry
~~~
$ oc login 

$ docker login -u $USERNAME -p $(oc whoami -t) {{docker_registry_route_hostname}}"
~~~

- Add policy
~~~
oadm policy add-role-to-user system:registry <user_name>
oadm policy add-role-to-user system:image-builder <user_name>
oadm policy add-role-to-user admin <user_name> -n openshift
~~~

- Push image/pull image
~~~
oc new-project test-docker-registry
docker pull busybox
docker tag docker.io/busybox docker-registry.cloudapps.example.com/test-docker-registry/busybox
docker push docker-registry.cloudapps.example.com/test-docker-registry/busybox
docker pull docker-registry.cloudapps.example.com/test-docker-registry/busybox
~~~

License
-------

BSD/MIT

Author Information
------------------

This role was created in 2017 by [Jooho Lee](http://github.com/jooho).

