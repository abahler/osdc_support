SSH Connections
===============

How to connect initially
------------------------
    * :ref:`Linux/OSX <ssh-linux>`
    * :ref:`Windows <ssh-windows>`


Troubleshooting your ssh connections
------------------------------------

Can not ssh into the Login nodes?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    #. Have you created/uploaded  your ssh key pairs via web console: :doc:`/console` ?
    #. Have you loaded your private key into a ssh agent?
       :ref:`Linux/OSX  <ssh-linux-agent>` :ref:`Windows <ssh-windows-puttygen>` ?

Can you not login to your VM's once created.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    #. Ensure that you have loaded your keys in your agent. :ref:`Linux/OSX <ssh-linux-agent>` or :ref:`Windows <ssh-windows-pageant>` ?
    #. Ensure that you have enabled ssh key forwarding :ref:`Linux/OSX <ssh-key-forwarding>` or :ref:`Windows <ssh-windows-putty>` ?
    #. :ref:`Is the ssh key showing up as forwarded on the login node <display-agent-keys>`?


.. _ssh-linux:

Linux/OSX
---------

Loading your private key into an SSH agent
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _ssh-linux-agent:

Adding a key to your ssh-agent on LINUX/OSX
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    Before you can load your key into ssh-agent you must ensure it is running.  On OSX this is automatic, on Linux you may need to do it manually or with a :ref:`script <ssh-agent-script>`.
    To load your key into the ssh-agent simply run ``ssh-add ~/.ssh/keyname``. If you password protected your private key you will be asked to enter the password.   By default keyname will be either ``~/.ssh/id_dsa`` or ``~/.ssh/id_rsa``.  You will most likely need to run this  command each time you start a terminal/cli session.
    If you are on OSX you can add this key to your OSX key ring by running ``ssh-add -k``. If you password protected your private key you will be asked this once for the password.  Everytime you open a new terminal window, OSX will auto populate the ssh keys you saved via ``ssh-add -k``.
    You can view your currently loaded keys with :ref:`ssh-add -l <display-agent-keys>`.

.. _ssh-agent-script:

Script to Auto start the ssh agent
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    If you are using a Desktop Environment (KDE/GNOME/XFCE) then it most likely is handling your ssh-agent/keyring.  However if yours does not, or your ssh key exists on an another server you can use the below script in your ``.bashrc`` file to auto start the agent.  Once started you still need to :ref:`add your keys <ssh-linux-agent>`.  If you are not using bash as your shell then you will need to modify as needed to fit your preferred environment.  You may also wish to try `keychain <http://www.funtoo.org/wiki/Keychain>`_.

.. code-block:: bash

    SSH_ENV="$HOME/.ssh/environment"
    alias ssh="ssh -A"

    function start_agent {
        echo "Initializing new SSH agent..."
        /usr/bin/ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
        echo succeeded
        chmod 600 "${SSH_ENV}"
        . "${SSH_ENV}" > /dev/null
        /usr/bin/ssh-add;
    }
    # Source SSH settings, if applicable
    if [ -f "${SSH_ENV}" ]; then
         . "${SSH_ENV}" > /dev/null
         #ps ${SSH_AGENT_PID} doesn't work under cywgin
         ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || start_agent;
    else
         start_agent;
    fi

.. _display-agent-keys:

Showing keys loaded into your agent on Linux/OSX/etc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Running the ``ssh-add -l``  command will display all keys currently loaded into your ssh agent.  Run this command from a shell (if not using putty) before ssh'ing into the login node to confirm that your key is properly loaded. Run it again once you have ssh'ed into the login node to confirm the key has properly forwarded.  If you do not see the key showing up on the login node, you will not be able to access your started Virtual Machines.
    Example Output

.. code-block:: bash

    #ssh-add -l
    1024 1a:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff:f1 /Users/JohnSmith/.ssh/id_dsa (DSA)`

.. _ssh-key-forwarding:

Enabling SSH key forwarding
~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Once your agent is configured you need to enable forwarding.  You can use any one of the below methods.

    * Open the ``ssh_config`` file located globally at ``/etc/ssh/ssh_config`` or locally at ``~/.ssh/config``. If this file does not exist under ``~/.ssh/`` then create it.  Add the following line ``ForwardAgent yes`` to this file.  All new connections will use forwarding.
    * When ssh'ing to the login node, use the ``-A`` flag.  This turns on forwarding on a case by case basis.  IF you have multiple login nodes that you are transversing, you will need to use the ``-A`` flag for all hops.  Example: ``ssh -A JohnSmith@sullivan.opensciencedatacloud.org``
    * Alias ``ssh -A`` as ``ssh`` via your shells preferred method.  On bash you can ``ALIAS ssh='ssh -A'``.


.. _ssh-windows:

Windows
-------

.. _ssh-windows-puttygen:

Convert OpenSSH key to Putty ppk format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Pageant.exe uses a different format then openssh for its keys.  We will need to convert the key to the ppk format.

    #. Start PuttyGen.exe
    #. Click ``Conversions``, then Click ``Import Key``
    #. Select the key you created and saved.  A screen will update
       extracting details from your key.  If your key is passworded you will need to manually enter the pass phrase next to  ``Key passphrase`` and ``Confirm passphrase``.
    #. Click ``Save private key``
    #. This will now save the private key in a format understandable by Pageant
        .. figure:: _static/puttygen.png
            :alt: PuttyGen.exe main screen
            :align: center


.. _ssh-windows-pageant:

Start Pageant
~~~~~~~~~~~~~
    #. Start Pageant.exe
    #. If the key is not listed in ``Pageant Key List``, Click ``Add``, then add the ppk file that you created ref:`above <ssh-windows-puttygen>`.  If it is already listed simply minimize Pageant.
        .. figure:: _static/pageant.png
            :alt: Pageant.exe import screen
            :align: center

.. _ssh-windows-putty:

Configuring Putty to use SSH Key Forwarding and Pageant
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    #. Open Putty
        .. figure:: _static/putty.png
            :alt: Putty.exe main screen
            :align: center
    #. Set ``Host Name (or IP address)`` to the hostname of the target login server provided to you. Port will be the default ``22``
    #. On the left side is a tree of available options called ``Category``.  Locate ``Connection`` and expand it, and select ``Data``.  Then enter your OSDC username in the ``username`` field.
    #. Return to the ``Connection`` category, locate  ``SSH`` and expand it then select Auth.
    #. Make sure the checkboxes for "Attempt authentication using Pageant" and "Allow agent forwarding” are selected.  Select them if not
        .. figure:: _static/putty-config-auth.png
            :alt: Putty.exe config auth screen
            :align: center
    #. Return to the ``Session`` category and enter a name for this session under ``Saved Sessions``, then click save.  From now on you need only ``Load`` this session to have all the proper settings preset.


