Ansible role: "Red Hat JBoss EAP" [![Build Status](https://travis-ci.org/Maarc/ansible-role-redhat-jboss-eap.svg?branch=master)](https://travis-ci.org/Maarc/ansible-role-redhat-jboss-eap) [![Galaxy](https://img.shields.io/badge/galaxy-maarc.rh--jboss--eap-blue.svg?style=flat)](https://galaxy.ansible.com/Maarc/rh-jboss-eap)
=================================


Description
-----------

Advanced Ansible role that manages [Red Hat JBoss Enterprise Application Platform (EAP) 6 or 7](https://access.redhat.com/documentation/en/red-hat-jboss-enterprise-application-platform/) instances.

Core implemented features in this role:

- multiple parallel versions and profile support
- multiple Red Hat JBoss EAP instances per host
- graceful orchestration and shutdown (prerequisite for rolling updates)
- application deployment using nexus or local files
- configuration of the Red Hat JBoss EAP instances using the CLI

Please have a look at [this example](https://github.com/Maarc/ansible_middleware_soe) showing how to easily operate Red Hat JBoss middleware products using this role.


Requirements
------------

This role has been tested on Ansible 2.0.2.0 and 2.1.1.0. It requires Red Hat Enterprise Linux 7.


Dependencies
------------

The "Maarc.rh-jboss-common" role is required. It could be imported as follows:

    ansible-galaxy install Maarc.rh-jboss-common -p roles

or

    ansible-galaxy install -r requirements.yml -p roles


Installation
------------

    ansible-galaxy install Maarc.rh-jboss-eap -p roles


Role Variables
--------------

*General variables*

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `download_dir` | `/tmp` | Directory containing all required middleware binaries on the managed remote host. Mandatory |
| `jboss.user` | `jboss` | Linux user name used for running EAP |
| `jboss.group` | `jboss` | Linux group name used for the `jboss.user` |
| `jboss.group_id` | `500` | Linux group id taken for `jboss.group` |
| `jboss.user_home` | `/opt/jboss` | Linux home directory for `jboss.user`  |


*Instance specific variables*

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `jboss_eap_instance_name` | `default` | Name of the separate running Red Hat JBoss EAP instance. Mandatory |
| `jboss_eap_instance_admin_user` | `redhat` | Red Hat JBoss EAP admin user name. Mandatory |
| `jboss_eap_instance_admin_password` | `ba2caa9378fa898f1dea88804abe52b4` | Red Hat JBoss EAP admin password ("redhat123!") hashed according to HEX( MD5( username ':' realm ':' password)). Mandatory |
| `jboss_eap_instance_admin_groups` | empty | Red Hat JBoss EAP admin user groups |
| `jboss_eap_golden_image_name` | empty | Name of the used Red Hat JBoss EAP golden image. Mandatory |
| `jvm_xm` | `512` | Value for the xms and xmx (both are set equal) in MB  |
| `jboss_eap_instance_port_offset` | `0` | Port offset for the JBoss EAP instance  |
| `jboss_eap_instance_cli_default_port` | `8888` | Port used only during updates using the CLI (port should be available) |
| `jboss_eap_instance_standalone_file` | `standalone.xml` | Name of the used standalone XML file |


*Usage of CLI files for the JBoss EAP configuration*

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `cli_list` | `{ }` | List of CLI files to be used for the configuration |
| `cli_dir` | empty | Local directory containing the CLI files in cli_list. Mandatory if `cli_list` is not empty |


*Java application deployments per file copy (direct)*

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `app_list` | `{ }` | List of the Java applications to be deployed |
| `app_dir` | empty | Local directory containing the files listed in app_list. Mandatory if 'app_list' is not empty |


*Java application deployments per over Nexus*

| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `app_mvn_list` | `{ }` | List of the nexus/maven applications to be deployed |
| `nexus_user` | empty | Nexus user name. Mandatory if 'app_mvn_list' is not empty. |
| `nexus_password` | empty | Nexus user password. Mandatory if 'app_mvn_list' is not empty. |
| `nexus_host` | empty | Nexus host name or IP. Mandatory if 'app_mvn_list' is not empty. |
| `nexus_repository_id` | empty | Nexus repository id (e.g. "releases"). Mandatory if 'app_mvn_list' is not empty. |


Example Playbook
----------------

Here is a playbook creating three JBoss EAP instances on every host in "jboss-group":

    - hosts: "jboss-group"
      roles:
        # JBoss EAP 7 instance for the ticket-monster application
        - {
            role: "Maarc.rh-jboss-eap",
            jboss_eap_golden_image_name: "jboss-eap-6.4.8_GI",
            jboss_eap_instance_name: "ticket_monster",
            jboss_eap_instance_standalone_file: "standalone-full-ha.xml",
            jboss_eap_instance_port_offset: 0,
            app_list: { "ticket-monster.war" },
            cli_list: { "add_datasource.cli", "add_mod_cluster_6.cli"},
        }
        # JBoss EAP 7 instance for the petclinic application
        - {
            role: "Maarc.rh-jboss-eap",
            jboss_eap_golden_image_name: "jboss-eap-7.0.1_GI",
            jboss_eap_instance_name: "petclinic",
            jboss_eap_instance_standalone_file: "standalone-full-ha.xml",
            jboss_eap_instance_port_offset: "1000",
            app_mvn_list: [ { g: "com.redhat.jboss", a: "petclinic.war", v: "1.0", e: "war" } ],
            cli_list: { "add_datasource.cli", "add_mod_cluster_7.cli"}
        }
        # JBoss EAP 7 instance for the jenkins application
        - {
            role: "Maarc.rh-jboss-eap",
            jboss_eap_golden_image_name: "jboss-eap-7.0.1_GI",
            jboss_eap_instance_name: "jenkins",
            jboss_eap_instance_standalone_file: "standalone-full-ha.xml",
            jboss_eap_instance_port_offset: 2000,
            app_list: { "jenkins.war" },
            cli_list: { "add_datasource.cli", "add_mod_cluster_7.cli"},
        }


Structure
---------

- `defaults/main.yml` centralize the default variables that could be overridden
- `tasks/main.yml` coordinate the execution of the different tasks
- `tasks/00__prepare.yml` check and create the linux user, group and home directory for the JBoss instance.
- `tasks/01__copy_and_unpack.yml` download the selected golden image in the `download_dir` and unzip it
- `tasks/02__configure.yml` used to check potential changes to the existing configuration (step 02) and to conduct the changes if necessary (step 04)
- `tasks/03__graceful_removal.yml` gracefully stops and removes the current running instance if necessary (based on the outcome of step 02)
- `tasks/05__register_service.yml` register the instance as a linux service
- `vars/main.yml` centralize some convenience variables that should not be overridden


License
-------

[Apache 2.0](./LICENSE)


Author Information
------------------

* [Marc Zottner](https://github.com/Maarc)
* [Roeland van de Pol](https://github.com/roelandpol)
