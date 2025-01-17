*********************************
Toolkit Installation Quick Start
*********************************

These instructions are for the full Toolkit installation on CentOS 7. For other perfSONAR installation options, see :doc:`install_options`.

#. Install CentOS 7 using your preferred method from https://centos.org

#. Login to the host and become sudo::

        sudo -s

#. Run the following commands::

        yum install epel-release http://software.internet2.edu/rpms/el7/x86_64/latest/packages/perfsonar-repo-0.11-1.noarch.rpm
        yum clean all
        yum install perfsonar-toolkit

#. Exit sudo and become sudo again to trigger login prompt::

        exit
        sudo -s

#. You will be prompted to create a user and password that can be used to administer the host through the web interface. Follow the prompts to complete this step.
    .. image:: images/install_quick_start-first-login-prompt.png
#. Open **http://<hostname>** in a web browser where **<hostname>** is the name or address of your host
#. Click on **Edit** (A) in the host information section of the main page or **Configuration** (B) button in the right-upper corner and login as the web administrator user created in the previous step
    
    .. image:: images/install_quick_start-web-admin-info1.png
#. On the page that loads, enter the requested information in the provided fields. In order to save **Administrative Information** you will be required to agree to the perfSONAR `Privacy Policy <https://www.perfsonar.net/about/privacy-policy/>`_. Tick the Privacy Policy checkbox to accept it. Click **Save** when you are done. 

    .. image:: images/install_quick_start-web-admin-info2.png
    .. seealso:: For more information on updating administrative information see :doc:`manage_admin_info`
#. You are now ready to add some regular tests. Click on **Configure tests** in main page.

    .. image:: images/install_quick_start-main-page.png
#. On the page that loads click on the **+Test** button too choose and add the test type you would like.

    .. image:: images/install_quick_start-add-test.png
#. A drop-down list shows to choose test type. Click on a selected test type you would like to add. 

    .. image:: images/install_quick_start-add-test-type.png
#. You will now be prompted for test parameters. Enter a human-readable description of the tests and change any parameters you desire. In general the defaults will be fine for most cases.

    .. image:: images/install_quick_start-add-test-params.png
#. You now need to select other test members to test against. Go to section **Test members** in the same page. 

    .. image:: images/install_quick_start-add-test-members-section.png
#. You may add test members explicitly adding a host (A) or selecting **browse communities** and browsing the list (B). When you are done entering host details, hit **Add host** to add a test member to the list of hosts.

    .. image:: images/install_quick_start-add-test-members-host.png
#. Click **OK** to save test definition and close test configuration window. Then click the **Save** button at the bottom of the screen to apply your changes.
    .. seealso:: For more information on adding regular tests see :doc:`manage_regular_tests`
#. After some time you may view the results of your tests in section **Test Results** in the main page.

    .. image:: images/install_quick_start-test-results.png

    .. warning:: It will take time for data to be collected and display on the graphs. For throughput data this may be several hours depending on the test interval. For all other test types, you should see data within 30 minutes.
    .. seealso:: For more information on using the graphs :doc:`using_graphs`


