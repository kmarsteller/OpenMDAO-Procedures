Building OpenMDAO-ready Machines on Amazon EC2
===============================================

Building OpenMDAO-ready machines on Amazon EC2 can be daunting.  Here is a checklist to make the
process a little bit less difficult.

* **Log in to Amazon Web Services under the openmdao account.**
  
  Enter your own username and password.  If you don't HAVE a username or password, contact Keith or
  Justin to make an account for you.  If you don't have a copy of ``lovejoykey.pem``, get one from
  a team member or download it from the EC2 site.

* **Start an instance**

  1. From the **EC2 Dashboard-->Instances**, click the **Launch Instance** button.

  2. When the **Request Instances Wizard** comes up, choose the **Quick Start** tab to get to 
     Amazon-approved Windows AMIs (Amazon Machine Images).  Choose the Windows version that suits you. 
     (For Linux,  look in `Community AMIs` for something other than SUSE; we use Ubuntu.)

  3. Choose the instance type.  

     For 64-bit OS, choose Large (``m1.large``).  

     For 32-bit OS, choose High-CPU Medium (``c1.medium``).

  4. On the next page of **Instance Details**, leave everything at the default setting except:
     "check the box to  enable Termination Protection."

  5. Next, give the machine a name that will help you identify it (using OS, architecture, etc.).

  6. On the next page, choose **lovejoykey** from the existing key pairs.

  7. On the next page, choose **Windows security group** if you're standing up a Windows box;
     otherwise, choose the default group.

  8. Review your settings and click **LAUNCH**. Now the VM you've created will be up and running, and
     you can configure it for OpenMDAO.


  **WINDOWS**

  * Follow the instructions above to create an instance. 

  * Then establish a **Remote Desktop Connection** into it.  

    You cannot SSH into the instance because Windows has no SSH by default. On EC2 there's a step to get
    a password for the Windows instance before you can connect. So you'll have to do that. The user name
    will be **Administrator**. 
    
    Once you're in, do the following.

  * Get Chrome, the Google browser.  

    You'll notice that EC2 Windows installs are RIDICULOUSLY locked down and that Internet Explorer is
    so lousy with lockdowns that you'll want to hit somebody.  Get Chrome here: 

	http://www.google.com/chrome/eula.html

  * Download and install Cygwin installer:  

	http://www.cygwin.com/install.html

    Make sure that while installing Cygwin you search the installables. Mark for installation:

    -  vi  (and any other editors)
    -  OpenSSH (OpenSSH setup will be discussed in detail below.)

  |  

  * Install Python 2.7 installer:

    -  Download from here: 

       http://python.org/download/

    -  Adjust the path:

       add ``C:\PythonXX\`` to PATH by going to **Control Panels** and editing **Env. Variables**

  * SSH setup:

    Setting up SSH on Windows, especially EC2, can be frustrating. And you can have it just right, 
    but the image may not work anyway when restarted. So you need to follow these instructions.

    - Open a Cygwin command prompt and enter the following commands:

      ::

        chmod +r /etc/passwd
        chmod +r /etc/group
        chmod 755 /var

    - Configure OpenSSH  

      You installed Cygwin above and made sure to install OpenSSH. Now we have to configure that for this
      machine.  

     * Open a Cygwin prompt and at the command line type ``ssh-host-config`` to start the configuration.  

     * From there, you will be prompted to answer some setup questions.   

       Say `yes` to everything that wants a yes/no answer.

     ::


      +privilege separation (yes), 
      +new local account 'sshd' ? (yes)
      +creating it as a service (yes) 
      +The answer to the question "Enter the value of CYGWIN for the daemon", is "ntsec".
      +The privileged account username by default will be "cyg_server." DO NOT ACCEPT THIS. Change this to "root" or the ssh won't work. 
       Give "root" the same password as everything else that we use for OpenMDAO.

     * When setup completes, hopefully with no errors, run the ssh daemon that you just configured by executing
       the command ``net start sshd``.

     * Double-check that the sshd is running.  Under the Services panel, you should see sshd running as a
       service.  If you see sshd in the services list, but it's idle, start it. If it won't start, something's
       wrong.

     * Go to the Windows Firewall and create a new rule to open TCP port 22 for SSH access.

     * Using Cygwin, go to the Administrator home directory (simply type ``cd``) and create a ``.ssh`` directory
       (``mkdir .ssh``).  

       - Go into the new dir and create a file called ``authorized_keys``.  

       - Into the ``authorized_keys`` file, paste the contents of all the keys for machines that want to log into this VM. (Grab this
	 info from the same file on an existing machine if you don't know what to do.)

       How you know it works: From a command line on your local machine, you should be able to type ``ssh
       Administrator@(EC2address)``, and it should log into that machine without having to type in a password. 
       (This will work only if the key from your local machine is in the keys file above.)

  * Git:

    You need Git. How else are you going to ``git OpenMDAO``? 

      http://code.google.com/p/msysgit/downloads/list?can=3&q=official+Git


  * gcc and gfortran:

    You're going to need compilers, namely mingw32 (for Fortran and C++). You can find mingw32 here:  

      http://sourceforge.net/projects/mingw/files  

    You must do the following when installing it: 

    1. Check the C++ compiler installation option to get g++ (required to run OpenMDAO).

    2. Create a file in the Administrator home directory called ``pydistutils.cfg`` that contains
       the following line: 

       ::

	 [build_ext] compiler=mingw32 

    3. Make sure to put the ``bin`` directory of the mingw32 install in your PATH.

  * numpy:

    Download the installer that corresponds to your Python version and run it.


      http://sourceforge.net/projects/numpy/files/

  * scipy:

    Download the installer that corresponds to your Python version and run it.


      http://sourceforge.net/projects/scipy/files/

  * matplotlib:

    Download the installer and run it.

      http://sourceforge.net/projects/matplotlib/files/matplotlib/matplotlib-1.1.0/

  If you did a test install of OpenMDAO, get rid of any ``.openmdao`` dir. Make sure that neither
  account's home directory contains a ``.openmdao`` directory.

  **LINUX**

  On Linux, things are much easier. At the very start, you'll launch an instance of a machine.  Once that
  instance is launched and configured properly (after you follow the below instructions), then you'll save
  an image of that machine. From then on out, you'll be able to spawn a bunch of instances from that one
  machine image.

  * So to start, simply choose the appropriate Linux distribution and the architecture (32- or 64-bit),
    and go.  

  * Once the instance is up and running, select its checkbox and choose **connect** from the options
    menu to get the address.

    Connecting from the command line is much simpler because SSH is already up and running on these
    machines.  The `superuser` name on these machines is **ubuntu**.  Simply type the following  (in
    a place where you can see the ``lovejoykey.pem``):

    ::

      ssh -i lovejoykey.pem ubuntu@<instance name with dashed numbers>.compute-1.amazonaws.com

  * Once you have SSHed into the machine, do the following:

    ::

      sudo apt-get update
      sudo apt-get install git
      sudo apt-get install gcc
      sudo apt-get install gfortran
      sudo apt-get install python-numpy
      sudo apt-get install python-scipy
      sudo apt-get install python-matplotlib
      sudo apt-get install python-dev
      sudo apt-get install python-markdown (to fulfill a requirement for EC2 images)
      sudo apt-get install openjdk-7-jre
      sudo apt-get install xvfb
      sudo apt-get watchdog
      sudo apt-get install firefox

      Use firefox to go and download the google chrome installation package.  Then install that with:
      dkpg -i [downloaded_chrome_package_name]


  * ``.openmdao`` dir:

    Once you've gotten everything configured and you've run OpenMDAO on the machine, make sure that you delete the root account's ``.openmdao`` directory.

* **Making an AMI image from an instance is the next and final step.** 

  An AMI is a machine image from which many instances can be spawned.  Machine images can be public
  or private, but they are private by default.  

  -  In the Instances screen, check the box next to the desired image; then choose
     **Make Amazon Machine Image (AMI)**.  

  -  Be sure to give the image a descriptive name using the format ``<Platform>_<architecture>_<python version>_OpenMDAO``, for example: ``Windows2008_64bit_py27_OpenMDAO``.  

  -  After waiting a bit, you'll finally have an image from which you can spawn multiple instances. 
     Good luck.
