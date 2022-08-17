# Installation of ArcGIS Enterprise using Ansible

This project is fucused on the installation of ArcGIS Enterprise components. This does not include the configuration of ArcGIS Enterprise base deployment.

This repo will help you to install ArcGIS Enterprise components on your testing machines.

For more defails for provisioing of ArcGIS Enterprise base deployment, see the [comming soon repository](https://atzhit7/arcgis-system-deploy).
That repository include the provisioning steps.

# Feature

I choose [Ansible](https://docs.ansible.com/ansible/2.9_ja/index.html) as a provisioning tool.
Ansible is used as a provisioning tool to install ArcGIS Enterprise because Ansible is simple, agentless.

This project includes:

    - ArcGIS Data Store installation.
        
    - ArcGIS Server installation.

    - Portal for ArcGIS installation.

There is no ArcGIS Web Adaptor role, you can prepare the web adaptor role or other reverse proxy server(i.e nginx).
The minimum configuration sample of nginx is [here](https://github.com/atzhit7/nginx-arcgis).

# Requirements

1. Prepare the following ArcGIS Enterprise installers.

* ArcGIS Data Store installer

* ArcGIS Server core installer (English installer)

    As the web gis system, the localized resouces must not used as the result of that applied localized resources.

* Portal for ArcGIS Installer

* Portal for ArcGIS Web Style installer


2. Prepare the target instances to install ArcGIS Enterprise.

**NOTE: if running Ansible locally, please check the network settings. Ansible provisioner will run through winrm and ssh(linux).**

# Usage

1. Clone this repo
1. Edit variables for your case.
    
    Variables to edit are:
    - vars_arcgisportal.yaml
    - vars_arcgisserver.yaml
    - vars_arcgisdatastore.yaml

    At least variables to edit are here (with all Uppercase WORDS except for shell arguments.):
    The sample variable yaml file for Portal for ArcGIS is here:

    ```yaml
    # common vars
    arcgis_version: VERSION YOU WANT TO INSTALL(ie. 10.9.1, 11.0. This is used in extract directory)
    arcgis_service_account_group: ARCGIS SERVICE ACCOUNT GROUP FOR LINUX
    arcgis_service_account: ARCGIS SERVICE ACCOUNT
    arcgis_service_account_pw: ARCGIS SERVICE ACCOUNT ENCRYPTED PASSWORD

    # for Windows
    win_arcgis_component: PortalForArcGIS
    win_arcgis_webstyle: ArcGISWebStyles
    win_product_id: '{CHECK SOFTWARE PRODUCTCODE}'
    # product_id is listed here: https://support.esri.com/en/Technical-Article/000013200
    win_remote_temp_dest_dir: FOR COPY-DESTINATEION DIRECCTORY
    win_remote_extract_dest_dir: FOR EXTRACTION OF INSTALLERS DIRECTORY
    win_copy_local_files:
    - PORTAL FOR ARCGIS INSTALLER FILE PATH
    - PORTAL FOR ARCGIS WEBSTYLE INSTALLER FILE PATH
    win_install_dir: INSTALL DIRECTORY
    win_portal_dir: CONTENT DIRECTORY
    win_install_log_dir: INSTALLATION LOG DIRECTORY
    win_install_log_path: "{{ win_install_log_dir }}\\arcgis-portal.log"

    win_setup_msi_path: "{{ win_remote_extract_dest_dir }}\\ArcGIS {{ arcgis_version }}\\{{ win_arcgis_component }}\\SetupFiles\\setup.msi"
    win_setup_msi_with_log_arguments: "/qn ACCEPTEULA{{'='}}yes INSTALLDIR{{'='}}{{ win_install_dir }} CONTENTDIR{{'='}}{{ win_portal_dir }} USER_NAME{{'='}}{{ arcgis_service_account }} PASSWORD{{'='}}{{ arcgis_service_account_pw }} /L*V {{ win_install_log_path }}"
    win_setup_msi_argumentlist: "/i \"\"{{ win_setup_msi_path }}\"\" {{ win_setup_msi_with_log_arguments }}"
    win_setup_msi_process_command: "Start-Process 'msiexec' -ArgumentList \"{{ win_setup_msi_argumentlist }}\" -Wait"
    win_setup_webstyle_msi_with_log_arguments: "/qn ACCEPTEULA{{'='}}yes /L*V {{ win_install_log_path }}"
    win_setup_webstyle_msi_path: "{{ win_remote_extract_dest_dir }}\\ArcGIS 10.9.1\\ArcGISWebStyles\\SetupFiles\\setup.msi"
    win_setup_webstyle_msi_argumentlist: "/i \"\"{{ win_setup_webstyle_msi_path }}\"\" {{ win_setup_webstyle_msi_with_log_arguments }}"
    win_setup_webstyle_msi_process_command: "Start-Process 'msiexec' -ArgumentList \"{{ win_setup_webstyle_msi_argumentlist }}\" -Wait" 
    # service name parameter
    win_service_name: "Portal for ArcGIS"


    # for Linux
    remote_temp_dest_dir: /tmp/arcgis
    remote_extract_dest_dir: "/home/{{ arcgis_service_account }}/setup"
    arcgisports:
    - 7080
    - 7443
    copy_local_files:
    - PORTAL FOR ARCGIS INSTALLER FILE
    - PORTAL FOR ARCGIS WEBSTYLE INSTALLER FILE
    install_dir: INSTALL DIRECTORY
    setup_command: "./Setup -m silent --verbose -l yes -d {{ install_dir }}"
    setup_command_location: "{{ remote_extract_dest_dir }}/PortalForArcGIS/"
    manual_start_shell: "./startportal.sh"
    manual_start_shell_location: "{{ install_dir }}/arcgis/portal/"
    service_unit_path: "{{ install_dir }}/arcgis/portal/framework/etc/arcgisportal.service"
    service_unit_dest_path: /etc/systemd/system/arcgisportal.service
    # service name parameter
    service_name: arcgisportal
    # OPTIONS for REDHAT
    rhelaccount: YOUR REDHAT ACCOUNT
    rhelaccount_password: YOUR REDHAT ACCOUNT PASSWORD


    # OPTIONS for AWS
    installer_bucket_name: YOUR BUCKET NAME HOSTING INSTALLERS
    logs_bucket_name: YOUR BUCKET NAME for LOGS
    win_copy_s3_files:
    - PORTAL FOR ARCGIS INSTALLER FILE
    - PORTAL FOR ARCGIS WEBSTYLE INSTALLER FILE
    copy_s3_files:
    - PORTAL FOR ARCGIS INSTALLER FILE
    - PORTAL FOR ARCGIS WEBSTYLE INSTALLER FILE
    ```

    **RECOMMENDATION: The service account password should be encrypted (such as ansible vault) or I prepared the task to create service account with the password from AWS Parameter Store such as "win_createuser_using_ssm.yaml". If using password in vars file, please change the main.yaml under common role as the following:**

    ```yaml
    # current
    - include_tasks: win_createuser_using_ssm.yaml
        when:
        - ansible_os_family == "Windows"
    # after
    - include_tasks: win_createuser.yaml
        when:
        - ansible_os_family == "Windows"
    ```

    **NOTE: win_ prefix varibales are for windows hosts.**

1. Modify the inventory with your hosts.
    
    I prepared 2 samples about singlenode and multiple nodes for Windows.
    
    * arcgis-all-in-one.cfg
    * arcgis-three-nodes.cfg

    If target nodes are linux, please use ssh connection.

1. run the following shell to install:

    `ansible-playbook -i <inventory_file_path> <playbook_file_path>`

    case1 - you need only arcgisdatastore installation on your target host:

    `ansible-playbook -i <inventory_file_path> arcgisdatastore.yaml`

    case2 - you need arcgis server/portal for arcgis/arcgis server installation on your single target host:

    `ansible-playbook -i <inventory_file_path> arcgis-all-in-one.yaml`

    case3 - you need arcgis components on each target hosts(3 hosts):

    `ansible-playbook -i <inventory_file_path> arcgis-three-nodes.yaml`

## About the Ansible Tasks

The roles include:

* arcgisdatastore - ArcGIS Data Store installation role
* arcgisserver - ArcGIS Server installation role
* arcgisportal - Portal for ArcGIS installation role
* common - common tasks for ArcGIS installation such as service account creation, firewall enabled etc.
* general - sample os configuration such as yum update or sample chocolatery installation python or boto etc.

I created "main.yaml" in each roles is designed to "include" tasks.

## More challenges of my project

I create other repos for your ArcGIS Enterprise System build.

See the following repos:

* [Comming soon - Packer Build project to prepare vm instance]()

* [comming soon - Ansible and Packer project for AWS Golden Image]()

* [comming soon - Terraform and Ansible Project for deploy ArcGIS Enterprise base deployment]()

