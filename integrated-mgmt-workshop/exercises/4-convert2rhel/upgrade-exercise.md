Intergated Management Workshop: CentOS/RHEL migration and upgrade
-----------------------------------------------------------------

**Introduction**<br>
This use-case will focus on conversion and migration from older CentOS versions to RHEL 8.3 (latest version as of Feb 2020). While we only show this process for a few systems, however, it can be scaled to a larger number of physical, virtual or cloud hosts using content repos provided by [Red Hat Satellite](https://www.redhat.com/en/technologies/management/satellite) (included in [Red Hat Smart Management](https://www.redhat.com/en/technologies/management/smart-management)). The upgrade process will be driven with automation built and run using [Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible).

**Environment**
- Satellite 6.8, Ansible Automation Platform 2.9
- 3x CentOS 7 instances 
- 3x RHEL 7  instances

**Upgrade Scenario**
- Exercise: Convert CentOS 7 to RHEL 7, then upgrade to RHEL 8
    - Covers CentOS 7 to RHEL 7 conversion
    - RHEL 7 to RHEL 8 upgrade

Overview
-----------------------------------------------------------------

**Summary**<br>
- Remember, during initial environment setup, we created a backup of the instance data (in case a fallback or restore is needed. Better safe than sorry.)
- We will utilize an additional project in Ansible Tower, "Three Tier App / Dev", which will allow us to install (take a guess) a three tier application stack across the three CentOS nodes. Additionally, the project also provides a means to test/verify functionality of the application components, which we will perform pre RHEL conversion.
- Next, we employ the Convert2RHEL utility to convert the CentOS nodes to RHEL. There are many sources of information on this handy utility, here are several of note:
    - [How to convert from CentOS or Oracle Linux to RHEL](https://access.redhat.com/articles/2360841) (Jan 2021)
    - [Converting from CentOS to RHEL with Convert2RHEL and Satellite](https://www.redhat.com/en/blog/converting-centos-rhel-convert2rhel-and-satellite) (March 2020)
    - [Convert2RHEL: How to update RHEL-like systems in place to subscribe to RHEL](https://www.redhat.com/en/blog/convert2rhel-how-update-rhel-systems-place-subscribe-rhel) (Jan 2020)
- Verify functionality of the application stack post RHEL conversion.
- Lastly, we will perform an in-place upgrade RHEL 7 to 8 (WIP)

Things to consider if doing this in dev/test/stage-beta/prod:
- Commercial and/or in-house developed application version(s) support with the host OS
- Bootloader changes
- Network connection and network time synchonizations


| **A Note about using Satellite vs. Ansible Automation Platform for this...**<br>  | 
| ------------- | 
| Out of the box, Satellite 6 supports [RHEL systems roles](https://access.redhat.com/articles/3050101) (a collection of Ansible Roles) for a limited set of administration tasks. An Ansible Automation Platform subscription is need to execute more complex Ansible jobs, such as OS conversions and upgrades. Using these two solutions together ensures you have the best tool for the job for:<br>- Content Management (Satellite)<br>- OS Patching & Standardized Operating Environments (Satellite)<br>- Provisioning: OS, Infra Services and Applications/Other (Satellite and/or Ansible Automation Platform)<br>- Configuration of Infra and Apps (Ansible Automation Platform)<br><br>Reference: [Scope of Support for Ansible Components included with Red Hat Satellite 6](https://access.redhat.com/articles/3616041) |


Ok, let's get started...  

Exercise:
-----------------------------------------------------------------
**Login to your Satellite & AAP UI's**
- [Satellite 6.8 UI](https://student1-sat.guid.domain.com)
- [Ansible Automation Platform 2.9 UI](https://student1.guid.domain.com)

Note that in the following steps that are being performed on Tower, at any time, over on the Satellite console, review the registered hosts via clicking Hosts => All Hosts.  Refresh the Hosts page to see changes as they occur a result from the automation being peformed via Tower.

**Steps:**<br>
1. Login to Ansible Tower, From the Dashboard main menu item, click INVENTORIES => Workshop Inventory => HOSTS
    - Here, you will see we have RHEL and CentOS Nodes in a **static** Ansible inventory

2. Take CentOS node snapshot (optional, if you've already performed this step during initial setup then skip)
    - template CONVERT2RHEL / 01 - Take node snapshot

8. Query Satellite to get node-related details, set EC2 instance tags based on these details
    - template EC2 / Set instance tags based on Satellite(Foreman) facts

9. Update inventories via dynamic sources
    - before you run the dynamic inventory update templates, navigate to the Inventories location in Tower and review the following inventories:
      - ALL Development => HOSTS
      - CentOS7 Development => HOSTS
      - RHEL7 Development => HOSTS
      
    Note that these inventories have yet to be populated.  Also, while you are in each of these inventories, click on the "SOURCES" button and review how each of these dynamic source inventories are configured, taking note of the "SOURCE VARIABLES" section to gain an understanding on how the resultant hosts and groups for that particular inventory are populated.

    - template TOWER / Update inventories via dynamic sources
	  - Select "CentOS7" for Inventory To Update
      - select "Dev" for Choose Environment

    - template TOWER / Update inventories via dynamic sources
	  - Select "RHEL7" for Inventory To Update
      - select "Dev" for Choose Environment

10. Upgrade CentOS nodes to latest
    - template CONVERT2RHEL / 02 - Upgrade OS to latest release

11. convert2rhel process
    - template CONVERT2RHEL / 03 - convert2rhel
      - choose LE group to convert CentOS7_Dev
      - choose LE target RHEL7_Dev
      - with some pre-configuration, any combination is possible

12. Query Satellite to get node-related details, set EC2 instance tags based on these details
    - template EC2 / Set instance tags based on Satellite(Foreman) facts

13. Refresh/update inventories
    - repeat step 9.

> **EXTRA CREDIT - Convert2RHEL workflow template**
Create a workflow template incorporating the above standalone templates into a complete CentOS to RHEL conversion workflow!

>**EXTRA CREDIT - Infrastructure-as-Code "Choose Your Own Adventure"**
  - Fork Automated Smart Management repo to individual GitHub account
Before we begin, you'll need to fork the Automated Smart Management repo into your personal GitHub account.  If you do not have an individual GitHub account, you will need to create one to proceed. Utilization of a source code management (SCM) system is central to the "infrastructure as code" concepts put forth in this lab exercise, and in this case, GitHub is our SCM.

Once logged into [GitHub](https://github.com) navigate to the [Red Hat Partner Tech repo for Automated Smart Management](https://github.com/redhat-partner-tech/automated-smart-management) repo. Next, on the Automated Smart Management repo page, in the top, upper right of the page, click "Fork".  This will create a "forked" Automated Smart Management repo in your personal GitHub account.

[Switch the "Automated Management" project in Tower to utilize your newly cloned repo](https://github.com/your-github-username/automated-smart-management.git). The following files are some good places to start looking to see where you can adjust the Extra Vars instance tags to select/filter what particular instances that a job template/playbook gets run against:

`group_vars/control/inventories.yml`

`group_vars/control/job_templates.yml`

Once the updates are made, commit and push these changes to the cloned repo, followed by running the "SETUP / Tower" job template, which will propogate the changes to Tower itself.
...
