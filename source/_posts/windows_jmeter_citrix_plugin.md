---
photos:
- /img/posts/citrix.jpg
tags: [testing]
categories:
- [performance-engineering]
---

## GCP Windows Image with Jmeter and Citirx

The whole article base on read me file from CitirixPlugin and own user experience.

 

 

The Citrix JMeter plugin enable the load-testing Citrix XenApp exposed applications.

 

CitirxPlugin required:

    - Windows (administrator rights are necessary)
    - Java OpenJDK 8 or higher for x86 architecture or i586 equivalent version (not 64 bits!)
    - Jmeter - (version 5.2.1 or later), plugins-manager


### Configuration VM image

 
#### Step by step:

    - Connect to google console
    - In Compute Engine - VM instance - Create Instance
    - Choose proper Region, Zone and Machine Configuration
    - Boot disk with windows image.

Remember to choose one of the support by Citirix Plugin:

    - Windows  7, 8.1, 10
    - Windows Server 2012 R2, 2016, 2019

    
    - Add Network tag, interface and Custom Metadata
    - Click on create VM and after VM stared - set&remeber windows user/password


### Jmeter and Citirx Installation

 

#### Step by step:

    - Connect to created VM use set usr/psswd
    - Install 32-bits java
    - Install Jmeter and plugins-manager
    - Install CitrixPlugin via plugins-manager in jmeter
    - Install Citirix client on machine

 


Author: {% link "Aga" https://performance-engineering-solutions.com/team/index.html#Aga %}
