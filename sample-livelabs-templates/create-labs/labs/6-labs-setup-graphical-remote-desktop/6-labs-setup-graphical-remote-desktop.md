# Setup Graphical Remote Desktop

## Introduction
This lab shows you how to deploy and configure noVNC Graphical Remote Desktop on an Oracle Enterprise Linux (OEL) instance prior to capturing the custom image.

### Objectives
- Configure image for preserved static hostname
- Deploy NoVNC Remote Desktop
- Configure Desktop
- Add Applications Shortcuts to Desktop
- Optimize Browser Settings
- Enable VNC password reset

### Prerequisites
This lab assumes you have:
- An Oracle Enterprise Linux 7 (OEL) that meets requirement for marketplace publishing

## Task 1: Configure Preserved Static hostname
Follow steps below to establish a unique static hostname that will be enforced on any offspring from the image. Whenever possible, this one-time task should be performed prior to installing any product that will hardcode the hostname to various config/settings in the product. e.g. DB Listener, Weblogic, etc...

1.  As opc, run *sudo su -* to login as root

    ```
    <copy>
    sudo su - || (sudo sed -i -e 's|root:x:0:0:root:/root:.*$|root:x:0:0:root:/root:/bin/bash|g' /etc/passwd && sudo su -)

    </copy>
    ```

2. Create script */root/bootstrap/firstboot.sh* .

    ```
    <copy>
    mkdir -p /root/bootstrap
    cat > /root/bootstrap/firstboot.sh <<EOF
    #!/bin/bash
    # Copyright (c) 2021 Oracle and/or its affiliates. All rights reserved.

    ################################################################################
    #
    # Name: "firstboot.sh"
    #
    # Description:
    #   Script to perform one-time adjustment to an OCI instance upon booting for the
    #   first time to preserve a static hostname across reboots and adjust any setting
    #   specific to a given workshop
    #
    #  Pre-requisite: This should be executed as "root" user.
    #
    #  AUTHOR(S)
    #  -------
    #  Rene Fontcha, Oracle LiveLabs Platform Lead
    #
    #  MODIFIED        Date                 Comments
    #  --------        ----------           -----------------------------------
    #  Rene Fontcha    02/17/2021           Initial Creation
    #  Rene Fontcha    10/07/2021           Added routine to update livelabs-get_started.sh
    #  Rene Fontcha    02/11/2022           Added Google Chrome update
    #
    ###############################################################################

    # Preserve user configured hostname across instance reboots
    sed -i -r 's/^PRESERVE_HOSTINFO.*\$/PRESERVE_HOSTINFO=2/g' /etc/oci-hostname.conf

    # Preserve hostname info and set it for current boot
    hostnamectl set-hostname <host>.livelabs.oraclevcn.com

    # Add static name to /etc/hosts
    echo "\$(oci-metadata -g privateIp |sed -n -e 's/^.*Private IP address: //p')   <host>.livelabs.oraclevcn.com  <host>" >>/etc/hosts

    # Update "livelabs-get_started.sh"
    cd /tmp
    wget -q https://objectstorage.us-ashburn-1.oraclecloud.com/p/RcNjQSg0UvYprTTudZhXUJCTA4DyScCh3oRdpXEEMsHuasT9S9N1ET3wpxnrW5Af/n/natdsecurity/b/misc/o/livelabs-get_started.zip

    if [[ -f livelabs-get_started.zip ]]; then
      unzip -q livelabs-get_started.zip
      chmod +x livelabs-get_started.sh
      mv -f livelabs-get_started.sh /usr/local/bin/
      rm -f livelabs-get_started.zip
    fi

    # Update Google Chrome
    yum update -y google-chrome-stable

    EOF
    </copy>
    ```

3. Create and run script */tmp/s_host.sh* to replace all occurrences of *<host>* from above file with the short name (not FQDN, so no domain) you would like permanently assigned to any instance created from the image. It requires providing the input for short hostname as prompted. e.g. *edq* resulting in FQDN *`edq.livelabs.oraclevcn.com`*

    ```
    <copy>
    cat > /tmp/s_host.sh <<EOF
    #!/bin/sh
    # Copyright (c) 2019 Oracle and/or its affiliates. All rights reserved.

    echo "Please provide the short hostname (not FQDN, so no domain) you would like permanently assigned to any instance created from the image:"
    read s_host
    echo ""
    echo "The permanent/preserved FQDN will be \${s_host}.livelabs.oraclevcn.com"
    sed -i "s/<host>/\${s_host}/g" /root/bootstrap/firstboot.sh
    EOF
    chmod +x /tmp/s_host.sh
    /tmp/s_host.sh
    </copy>
    ```

    *Note:* If you need to set a specific FQDN to satisfy an existing product setup, manually edit */root/bootstrap/firstboot.sh* and update replace all occurrences of *`<host>`* and *`livelabs.oraclevcn.com`* accordingly.

4. Make script */root/bootstrap/firstboot.sh* executable, add soft link to */var/lib/cloud/scripts/per-instance* and run it

    ```
    <copy>
    chmod +x /root/bootstrap/firstboot.sh
    ln -sf /root/bootstrap/firstboot.sh /var/lib/cloud/scripts/per-instance/firstboot.sh
    /var/lib/cloud/scripts/per-instance/firstboot.sh
    hostname
    exit

    </copy>
    ```

## Task 2: Deploy noVNC
1.  As root, download and run the latest setup script. You will be prompted for the following input:

    - The *OS user* for which the remote desktop will be configured. *Default: Oracle*

    ```
    <copy>
    sudo su - || (sudo sed -i -e 's|root:x:0:0:root:/root:.*$|root:x:0:0:root:/root:/bin/bash|g' /etc/passwd && sudo su -)

    </copy>
    ```

    ```
    <copy>
    cd /tmp
    rm -rf ll-setup
    wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/Nx05fQvoLmaWOPXEMT_atsi0G7Y2lHAlI7W0k5fEijsa-36DcucQwPUn6xR2OIH8/n/natdsecurity/b/misc/o/setup-novnc-livelabs.zip -O setup-novnc-livelabs.zip
    unzip -o  setup-novnc-livelabs.zip -d ll-setup
    cd ll-setup/
    chmod +x *.sh .*.sh
    ./setup-novnc-livelabs.sh

    </copy>
    ```

2. Launch your web browser to the URL provided in output from above execution, then *Next*

    E.g.

    ```
    <copy>http://[your instance public-ip address]/livelabs/vnc.html?password=LiveLabs.Rocks_99&resize=scale&quality=9&autoconnect=true&reconnect=true</copy>
    ```

    ![](./images/novnc-landing-1a.png " ")

3. Click *Next*

    ![](./images/novnc-landing-1b.png " ")

4. Click *Next*

    ![](./images/novnc-landing-1c.png " ")

5. Click *Skip*

    ![](./images/novnc-landing-1d.png " ")

6. Click *Start Using Oracle Linux Server*

    ![](./images/novnc-landing-1e.png " ")

7. Click on *X* to close the *Getting Started* landing page

    ![](./images/novnc-landing-1f.png " ")

8. Right-Click anywhere in the desktop and select *Open Terminal*

    ![](./images/novnc-landing-1.png " ")

9. Run the following to configure and optimize the desktop

    ```
    <copy>
    /tmp/ll-setup/.config-gnome-desktop.sh
    </copy>
    ```

    ![](./images/novnc-landing-2.png " ")
    ![](./images/novnc-landing-3.png " ")

10. Double-Click on *google-chrome.desktop* icon, Keep *Make Google Chrome the default browser* checked, uncheck *Automatic Usage Statistics & Crash reporting* and click *OK*

    ![](./images/novnc-custom-chrome-1a.png " ")

11. Click on *Get Started*, on the next 3 pages click on *Skip*, and finally on *No Thanks*.

    ![](./images/novnc-custom-chrome-3a.png " ")
    ![](./images/novnc-custom-chrome-4a.png " ")
    ![](./images/novnc-custom-chrome-5a.png " ")
    ![](./images/novnc-custom-chrome-6a.png " ")

12. Click in the *Three dots* at the top right, then select *"Bookmarks >> Show bookmarks bar"*

    ![](./images/add-bookmarks-01.png " ")

13. Right-click anywhere in the *Bookmarks bar area*, then Uncheck *Show apps shortcuts* and *Show reading list*

    ![](./images/add-bookmarks-04.png " ")

14. Enter the URL below in the *Address* field and press *Enter* to access the *LiveLabs* homepage.

    ```
    <copy>http://bit.ly/golivelabs</copy>
    ```

    ![](./images/add-bookmarks-02.png " ")

15. Click on the *star* at the end of the *Address* field, then on *Add bookmark* to create a bookmark to *LiveLabs* homepage

    ![](./images/add-bookmarks-03.png " ")

16. Keep everything unchanged and click on *Done*.

    ![](./images/add-bookmarks-05.png " ")

17. Click in the *Three dots* at the top right, then select *Settings*

    ![](./images/add-bookmarks-06.png " ")

18. Scroll down to *On Startup* section, select *open a specific page or set of pages*, and select *Use current pages* or simply add the *LiveLabs* address you set earlier as bookmark.

    ![](./images/add-bookmarks-07.png " ")

19. Run the following from terminal session to initialize LiveLabs browser windows.

    ```
    <copy>
    $HOME/.livelabs/init_ll_windows.sh
    </copy>
    ```
20. If the *`desktop_app1_url`* and/or *`desktop_app2_url`* are applicable to the workshop, test with *chrome-window2* chrome profile to validate before proceeding to custom image creation.

    e.g. The example below is from the *DB Security - Key Vault* workshop

    ```
    <copy>
    user_data_dir_base="/home/$(whoami)/.livelabs"
    desktop_app1_url="https://kv"
    desktop_app2_url="https://dbsec-lab:7803/em"
    google-chrome --password-store=basic ${desktop_app1_url} --window-position=1010,50 --window-size=887,950 --user-data-dir="${user_data_dir_base}/chrome-window2" --disable-session-crashed-bubble >/dev/null 2>&1 &
    google-chrome --password-store=basic ${desktop_app2_url} --window-position=1010,50 --window-size=887,950 --user-data-dir="${user_data_dir_base}/chrome-window2" --disable-session-crashed-bubble >/dev/null 2>&1 &
    </copy>
    ```

21. Update *vncserver* startup script to add dependency(ies) on primary service(s) supporting Web Apps behind *`desktop_app1_url`* and/or *`desktop_app2_url`*. This will prevent premature web browser startup leading to *404-page-not-found-error* when the app requested is not yet ready.

    - Edit `/etc/systemd/system/vncserver_${appuser}@\:1.service` and append the dependent service(s) at the end of the starting with **After=**

    e.g. The example below is from the *EM Fundamentals* workshop, please substitute *oracle-emcc.service* in the block below with the correct service name relevant to your workshop before running it.

    ```
    <copy>
    sudo sed -i "/^After=/ s/$/ oracle-emcc.service/" /etc/systemd/system/vncserver_$(whoami)@\:1.service
    sudo systemctl daemon-reload
    cat /etc/systemd/system/vncserver_$(whoami)@\:1.service |grep After
    </copy>
    ```

    ![](./images/add-bookmarks-08.png " ")

    - Verify the output as shown above and confirm that the service dependency has been successfully added

22. Close all browser windows opened.

23. Right-Click anywhere in the desktop and Uncheck *Keep aligned*

    ![](./images/novnc-organize-desktop-1.png " ")

24. Right-Click anywhere in the desktop and select *Organize Desktop by Name*

    ![](./images/novnc-organize-desktop-2.png " ")
    ![](./images/novnc-organize-desktop-3.png " ")

25. Review *`$HOME/.bash_profile`* and confirm the presence of the following default code block (add if missing).

    ```
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
        . ~/.bashrc
    fi
    ```

26. After validating successful setup from URL displayed by above script, cleanup the setup directory "*/tmp/ll-setup*"

    ```
    <copy>
    rm -rf /tmp/ll-setup

    </copy>
    ```

## Task 3: Adding Applications Shortcuts to Desktop   
For ease of access to the workshop guide and desktop applications provided on the instance, the following shortcuts are configured by default:

- Get Started with your Workshop (launch workshop guide and relevant web apps if any)
- Google Chrome Browser
- Gnome Terminal

Follow the steps below to add any other desktop application shortcut relevant to your workshop. The example below is for illustration only as it features the already configured LiveLabs custom desktop application *Get Started with your Workshop*. As a result, use it as a reference and substitute accordingly for the relevant desktop application of your choice (e.g. SQLDeveloper, JDeveloper, Postman, etc...).

1. On the remote desktop, click on *Home > Other Locations*, then navigate to *`/usr/share/applications`* and scroll-down to find *Get Started with your Workshop*

    ![](./images/create-shortcut-1.png " ")

2. Right-click on *Get Started with your Workshop* and select *Copy to...*

    ![](./images/create-shortcut-2.png " ")

3. Navigate to *Home > Desktop* and Click on *Select*

    ![](./images/create-shortcut-3.png " ")

4. Double-click on the newly added icon on the desktop and click on *Trust and Launch*

    ![](./images/create-shortcut-4.png " ")
    ![](./images/create-shortcut-5.png " ")

## Task 4: Configure Additional Desktop Apps for Auto-Start on VNC Startup   
LiveLabs compute instance are password-less and only accessible optionally via SSH keys. As result it's important to adjust session settings to ensure a better user experience. By default the dedicated LiveLabs custom desktop application *Get Started with your Workshop* is setup to automatically launch web browser session(s) on:

- First half (left) of the screen preloaded with the workshop guide
- Second half (right) of the screen preloaded with up to 2 tabs with relevant web apps if applicable

If there are no WebApps used in the workshop, configure *Startup Programs* for another application such as *SQL Developer* to open up on the right next to the workshop guide on *VNC* startup

1. From the same Terminal window, run the following command to open *Startup Programs* configuration.

    ```
    <copy>
    gnome-session-properties
    </copy>
    ```

2. Fill in the details as shown below and click *Add* to add *SQL Developer* to the list of applications to be started automatically on *VNC* Startup

    - Name

    ```
    <copy>SQL Developer</copy>
    ```

    - Command

    ```
    <copy>sqldeveloper</copy>
    ```

    - Comment

    ```
    <copy>Launch SQL Developer on VNC Startup</copy>
    ```

    ![](./images/novnc-startup-prog-1a.png " ")

3. Restart *vncserver* to test.

    ```
    <copy>sudo systemctl restart vncserver_$(whoami)@\:1</copy>

    ```

    ![](./images/novnc-startup-prog-2a.png " ")

4. Wait for *Auto reconnect* to get back into the remote desktop

    ![](./images/novnc-startup-prog-3a.png " ")

    *Notes:* Don't worry if the browser window(s) is(are) not loaded as expected on VNC startup at the moment. The required instance metadata is not yet present on the host but will be injected at provisioning to cover the following.

    - `DESKTOP_GUIDE_URL` - *required*
    - `DESKTOP_APP1_URL` - optional
    - `DESKTOP_APP2_URL` - optional

    The following is an example from the *Boost Analytics Performance with Oracle In-Memory Database* workshop

    ![](./images/novnc-startup-prog-6a.png " ")

You may now [proceed to the next lab](#next).

## Appendix 1: Enable VNC Password Reset, and Workshop Guide and WebApps URLs injection for each instance provisioned from the image
Actions provided in this Appendix are not meant to be performed on the image. They are rather intended as guidance for workshop developers writing terraform scripts to provision instances from an image configured as prescribed in this guide.

Update your Terraform/ORM stack with the tasks below to enable VNC password reset and add workshop URLs for each VM provisioned from the image.

1. Add provider *random* to *main.tf* or and any other *TF* file in your configuration if you not using *main.tf*

    ```
    <copy>
    terraform {
      required_version = "~> 0.13.0"
    }

    provider "oci" {
      tenancy_ocid = var.tenancy_ocid
      region       = var.region
    }

    provider "random" {}
    </copy>
    ```
2. Add the following variables to *variables.tf* and *schema.yaml*.

    - `desktop_guide_url`
    - `desktop_app1_url`
    - `desktop_app2_url`

    The example below is from the *DB Security - Key Vault* workshop

    - variables.tf

    ```
    <copy>
    variable "desktop_guide_url" {
      default = "https://oracle.github.io/learning-library/security-library/database/advanced/workshops/main-key-vault"
    }

    variable "desktop_app1_url" {
      default = "https://kv"
    }

    variable "desktop_app2_url" {
      default = "https://dbsec-lab:7803/em"
    }
    </copy>
    ```

    - schema.yaml

    ```
    variableGroups:
      - title: General Configuration
        visible: false
        variables:
        - desktop_guide_url
        - desktop_app1_url
        - desktop_app2_url

    desktop_guide_url:
      type: text
      required: true
      title: "Workshop Guide"
      description: "Workshop Guide on noVNC Desktop"

    desktop_app1_url:
      type: text
      required: false
      title: "Application URL 1"
      description: "Application URL 1 on noVNC Desktop"

    desktop_app2_url:
      type: text
      required: false
      title: "Application URL 2"
      description: "Application URL 2 on noVNC Desktop"
      </copy>
      ```

3. Add a *random* resource in your *instance.tf* or any *TF* of your choice to generate a 10 characters random password with a mix of Number/Uppercase/Lowercase characters.

    ```
    <copy>
    resource "random_string" "vncpwd" {
      length  = 10
      upper   = true
      lower   = true
      number  = true
      special = false
    }
    </copy>
    ```

4. Add *`random_string`* result and the URL variables to the metadata property for resource *`oci_core_instance`*. This will store the random value generated above as part of the instance metadata and used on first boot to reset VNC Password. The URLs will be used to preload the workshop guide and webapps on the remote desktop on VNC startup

    ```
    <copy>
    metadata = {
      vncpwd            = random_string.vncpwd.result
      desktop_guide_url = var.desktop_guide_url
      desktop_app1_url  = var.desktop_app1_url
      desktop_app2_url  = var.desktop_app2_url
    }
    </copy>
    ```

5. Add the entry *`remote_desktop`* to your *output.tf* to provide the single-click URL for remote desktop access with auto resizable window and auto-login. Replace [instance-name] from the snippet below with your real instance name as provided the resource *`oci_core_instance`* block of *instance.tf*

    ```
    <copy>
    output "remote_desktop" {
      value = format("http://%s%s%s%s",
        oci_core_instance.[instance-name].public_ip,
        "/livelabs/index.html?password=",
        random_string.vncpwd.result,
        "&resize=scale&autoconnect=true&quality=9&reconnect=true"
      )
    }
    </copy>
    ```
6. Add output entry *`remote_desktop`* to your *schema.yaml* file

    ```
    <copy>
    outputGroups:
      - title: Resources Access Information
        outputs:
          - ${remote_desktop}

    outputs:
      remote_desktop:
        type: string
        title: Remote Desktop
        visible: true

    </copy>
    ```

7. Add an *ingress* rule to your *network.tf* to enable remote access to port *6080* when the VCN is created

    ```
    <copy>
    ingress_security_rules {
      protocol = "6"
      source   = "0.0.0.0/0"
      tcp_options {
        min = 6080
        max = 6080
      }
    }

    </copy>
    ```

8. Test out your ORM Stack and verify the output for *`remote_desktop`* as shown below

    ![](./images/orm-output.png " ")

9. From to the *Application Information Tab* as shown above, click on the single-click URL to test it out.

    ![](./images/orm-single-click-url.png " ")

    **Note:** Your source image instance is now configured to generate a random VNC password for every instance created from it, provided that the provisioning requests include the needed metadata storing the random string.

## Appendix 2: Removing Guacamole from a previously configured LiveLabs image

Prior to noVNC some images were configured with *Apache Guacamole*. If this applies to your image, proceed as detailed below to remove it prior to deploying noVNC

1.  As root, create and run script */tmp/remove-guac.sh*.

    ```
    <copy>
    sudo su - || (sudo sed -i -e 's|root:x:0:0:root:/root:.*$|root:x:0:0:root:/root:/bin/bash|g' /etc/passwd && sudo su -)

    </copy>
    ```

    ```
    <copy>
    cat > /tmp/remove-guac.sh <<EOF
    #!/bin/sh
    # Copyright (c) 2019 Oracle and/or its affiliates. All rights reserved.

    cd /etc/systemd/system

    for i in `ls vncserver_*.service`
      do
    systemctl stop $i
    done

    cd /tmp

    systemctl disable guacd tomcat
    systemctl stop guacd tomcat

    yum -y remove \
    	guacd \
        libguac \
        libguac-client-ssh \
        libguac-client-vnc \
    	tomcat \
        tomcat-admin-webapps \
        tomcat-webapps
    EOF
    chmod +x /tmp/remove-guac.sh
    /tmp/remove-guac.sh

    rm -rf /etc/guac*
    rm -f /tmp/remove-guac.sh
    rm -rf /opt/guac*
    rm -rf /etc/init.d/guac*
    cd
    </copy>
    ```

## Acknowledgements
* **Author** - Rene Fontcha, LiveLabs Platform Lead, NA Technology, September 2020
* **Contributors** - Robert Pastijn
* **Last Updated By/Date** - Rene Fontcha, LiveLabs Platform Lead, NA Technology, March 2022
