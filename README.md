Ansible role: "Red Hat JBoss EAP" [![Build Status](https://travis-ci.org/Maarc/ansible-role-redhat-jboss-eap.svg?branch=master)](https://travis-ci.org/Maarc/ansible-role-redhat-jboss-eap)
=================================


Description
-----------

Advanced Ansible role that installs and manages instances of [Red Hat JBoss Enterprise Application Platform (EAP) 6 or 7](https://access.redhat.com/documentation/en/red-hat-jboss-enterprise-application-platform/).

Core implemented features in this role:

- multiple parallel versions and profile support
- multiple Red Hat JBoss EAP instances per host
- graceful orchestration and shutdown (prerequisite for rolling updates)
- application deployment using nexus or local files
- configuration of the Red Hat JBoss EAP instances using the CLI

Please have a look at [this example](https://github.com/Maarc/ansible_middleware_soe) showing how to easily operate Red Hat JBoss middleware products using this role.



Requirements
------------

This has been tested on Ansible 1.9.4 and higher. It requires Red Hat Enterprise Linux 7.


Dependencies
------------

None.


Installation
------------

  ansible-galaxy install Maarc.rh-jboss-common




Role Variables
--------------




Dependencies
------------

-


Example Playbook
----------------

Here is a playbook creating three JBoss EAP instances on every host in "jboss-group":

    - hosts: "jboss-group"
      roles:
        - {
            # JBoss EAP 7 instance for the ticket-monster application
            role: "rh-jboss-eap",
            jboss_eap_golden_image_name: "jboss-eap-6.4.8_GI",
            jboss_eap_instance_name: "ticket_monster",
            jboss_eap_instance_standalone_file: "standalone-full-ha.xml",
            jboss_eap_instance_port_offset: 0,
            app_list: { "ticket-monster.war" },
            cli_list: { "add_datasource.cli", "add_mod_cluster_6.cli"},
          }
        - {
            # JBoss EAP 6 instance for the petclinic application (note: toggle role and app to deploy from Nexus)
            role: "rh-jboss-eap",
            jboss_eap_golden_image_name: "jboss-eap-7.0.1_GI",
            jboss_eap_instance_name: "petclinic",
            jboss_eap_instance_standalone_file: "standalone-full-ha.xml",
            jboss_eap_instance_port_offset: "1000",
            app_list: { "petclinic.war" },
            cli_list: { "add_datasource.cli", "add_mod_cluster_7.cli"},
            #app_mvn_list: [ { g: "com.redhat.jboss", a: "petclinic.war", v: "1.0", e: "war" } ],
          }
        - {
            # JBoss EAP 7 instance for the jenkins application
            role: "rh-jboss-eap",
            jboss_eap_golden_image_name: "jboss-eap-7.0.1_GI",
            jboss_eap_instance_name: "jenkins",
            jboss_eap_instance_standalone_file: "standalone-full-ha.xml",
            jboss_eap_instance_port_offset: 2000,
            app_list: { "jenkins.war" },
            cli_list: { "add_datasource.cli", "add_mod_cluster_7.cli"},
          }


Documentation
-------------




License
-------

[Apache 2.0](./LICENSE)


Author Information
------------------

* [Marc Zottner](https://github.com/Maarc)
* [Roeland van de Pol](https://github.com/roelandpol)
