
## Sample Ansible Playbooks for Cisco ACI using Ansible Core Modules ##

The playbooks contained within this repository are examples only.  They are intended to illustrate some basic functions against an ACI Fabric utilizing the ACI Ansible Core modules.

To get started right away, I have some purpose built Docker images that are useful for quick tests.

As usual, many of these public playbooks use the [DevNet](https://developer.cisco.com/) Always On Sandbox devices so you don't even need a lab!

[Ansible Server in Docker for Network Engineers](https://gratuitous-arp.net/ansible-server-in-docker-for-network-engineers/)

These playbook use the following ACI Modules:
- [aci_tenant – Manage tenants (fv:Tenant)](https://docs.ansible.com/ansible/latest/modules/aci_tenant_module.html#aci-tenant-module)
- [aci_vrf – Manage contexts or VRFs (fv:Ctx)](https://docs.ansible.com/ansible/latest/modules/aci_vrf_module.html#aci-vrf-module)
- [aci_rest – Direct access to the Cisco APIC REST API](https://docs.ansible.com/ansible/latest/modules/aci_rest_module.html)


REF: [Ansible Network Module Index](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html)


##### Main Playbooks ######
 - aci\_lab.yml      
	 	-- Basic starter Playbook to Query Tenants in an ACI Fabric
 - aci\_tenant_prompt.yml  
    -- Basic Playbook which prompts for tenant name and action and then displays all tenants in the ACI Fabric
 - aci\_new_vrf.yml  
    -- Playbook which queries Tenants and allows user to select the specific Tenant which requires a VRF action (add state=present, remove state=absent, or query (default), performs requested action and queries specified tenant for list of VRfs
    
##### Miscellaneous and Supporting Playbooks ######

Note: These are "works in progress"!

 - aci\_tenant_query.yml  
    -- Test Playbook
 - aci\_tenant_query2.yml  
    -- Test Playbook for tenant interactivce input
 - all\_host_vars.yml  
	 	-- Sample Playbook to display all variables for a host defined in the playbook namespace
 - port_blacklist.yml 
	 	-- Sample Playbook which takes a payload data YAML file (group\_vars/port_blacklist_payload_data.yml), builds the required payload to disable (blacklist) or enable interfaces using a Jinja2 template (templates/int\_enable\_disable.j2) and pushes that payload to the APIC.


```
(aci_build) Claudias-iMac:cisco_aci claudia$ tree
.
├── YAML-Payload
├── aci_lab.yml
├── aci_new_vrf.yml
├── aci_rest.yml
├── aci_rest_get.yml
├── aci_tenant_prompt.yml
├── aci_tenant_query.yml
├── aci_tenant_query2.yml
├── all_host_vars.yml
├── ansible.cfg
├── group_vars
│   ├── creds.yml
│   └── port_blacklist_payload_data.yml
├── host_vars
│   └── host_var_directory
├── hosts
├── logs
│   └── output_directory
├── payload
│   └── TEST123-port-blacklist.json
├── port-switch-inspector\ output.log
├── port_blacklist.yml
├── readme.MD
└── templates
    ├── aci_tenant_report_template.j2
    └── int_enable_disable.j2

5 directories, 21 files
(aci_build) Claudias-iMac:cisco_aci claudia$ 


```



#### Playbook Run Examples ####

> hosts file

```
root@0bd4731cb725:/ansible/cisco_aci# cat hosts
[local]
localhost ansible_connection=local

[aci]
sandboxapicdc.cisco.com
```

>  # ansible-playbook -i hosts aci\_tenant_prompt.yml


```
root@0bd4731cb725:/ansible/cisco_aci# ansible-playbook -i hosts aci_tenant_prompt.yml
Enter Tenant Name: [LAX_Tenant]: AMS_Tenant
Enter Tenant State (absent | present | query) [query]: present

PLAY [ACI Playbook to Perform User Prompted Action on Tenant] ****************************************************************************************************************

TASK [Set Date] **************************************************************************************************************************************************************
ok: [sandboxapicdc.cisco.com]

TASK [Add, Delete, or Query Tenant AMS_Tenant] *******************************************************************************************************************************
changed: [sandboxapicdc.cisco.com -> localhost]

TASK [debug] *****************************************************************************************************************************************************************
ok: [sandboxapicdc.cisco.com] => {
    "tenant_results": {
        "changed": true,
        "current": [
            {
                "fvTenant": {
                    "attributes": {
                        "descr": "Tenant AMS_Tenant Action present on 2019-02-24 22:00",
                        "dn": "uni/tn-AMS_Tenant",
                        "name": "AMS_Tenant",
                        "nameAlias": "",
                        "ownerKey": "",
                        "ownerTag": ""
                    }
                }
            }
        ],
        "failed": false
    }
}

TASK [Query all tenants] *****************************************************************************************************************************************************
ok: [sandboxapicdc.cisco.com -> localhost]

TASK [Display RAW APIC Results] **********************************************************************************************************************************************
skipping: [sandboxapicdc.cisco.com]

TASK [set_fact] **************************************************************************************************************************************************************
ok: [sandboxapicdc.cisco.com]

TASK [Tenants returned in Query ==>] *****************************************************************************************************************************************
ok: [sandboxapicdc.cisco.com] => {
    "tenantlist": [
        "common",
        "mgmt",
        "infra",
        "Heroes",
        "SnV",
        "API_Test",
        "production",
        "GDL_Tenant",
        "Mostafa_Tenant",
        "LAX_Tenant",
        "AMS_Tenant"
    ]
}

PLAY RECAP *******************************************************************************************************************************************************************
sandboxapicdc.cisco.com    : ok=6    changed=1    unreachable=0    failed=0

root@0bd4731cb725:/ansible/cisco_aci#
```





### Environment ##

The lab environment used for development and initial testing utilizes the DevNet APIC Sandbox.

The Ansible Control System was hosted on a Docker Cotainer for Ansible 2.4 (upgraded to Ansible 2.6 using the pip install command below)


```
 pip install ansible==2.6.8.0
```

##### Docker Container #####

https://hub.docker.com/r/cldeluna/xenial-ansible/





```
root@0bd4731cb725:/ansible/cisco_wlc# ansible --version
ansible 2.6.8
  config file = /ansible/cisco_wlc/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Nov 19 2016, 06:48:10) [GCC 5.4.0 20160609]
root@0bd4731cb725:/ansible/cisco_wlc#
```

##### Considerations ######

As of Ansible 2.5 significant changes were introduced to some well established methods (provider, username, password) for connecting to network devices.  In short, these [new methods](https://docs.ansible.com/ansible/latest/porting_guides/porting_guide_2.5.html#adding-persistent-connection-types-network-cli-and-netconf) allow for consistency in connecting to any type of target be it server or network device.  This is good for the many, to be sure, but will result in rework for those of us who have older playbooks.  We will also have to keep an eye out on our favorite modules to ascertain when they can support the new constructs.


[Ansible 2.5 Porting Guide](https://docs.ansible.com/ansible/latest/porting_guides/porting_guide_2.5.html)

>Top-level connection arguments will be removed in 2.9

>Top-level connection arguments like username, host, and password are deprecated and will be removed in version 2.9.

[IOS Platform Options](https://docs.ansible.com/ansible/latest/network/user_guide/platform_ios.html#using-cli-in-ansible-2-6)


##### Next Steps ######

- Using the REST module
- More complicated playbooks with validation

##### Disclaimer ######

*THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NON INFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.*
