
    #!div class="important" style="border: 2pt solid; text-align: center"
    '''''DEPRECATED, NO LONGER USED'''''

You might want to look at UploadFedoraContent instead.
# Uploading rpm packages into Spacewalk

----


In order to use Spacewalk, you'll need to upload rpm packages into it. At the moment there is no nice way to sync a yum repo with Spacewalk.  The following steps are covered in order to get your Spacewalk Server serving content.

        - [UploadContent#Creatinganewchannel Creating a new channel] to hold the rpm packages
        - [UploadContent#Downloadtherpmpackages Download the rpm packages] from a yum repo
        - [UploadContent#Uploadtherpmpackages Upload the rpm packages] into your Spacewalk Server


----
## Creating a new channel



In order to upload rpm packages into Spacewalk, you'll need to create a channel that will contain the packages.  There are two ways to create a new channel.  You can use the attached command line tool which uses the available XML-RPC API.

For example, to create a channel for Fedora 9:

    ./create_channel.py --label=fedora-9-i386 --name "Fedora 9 32-bit" --summary "32-bit Fedora 9 channel"
    
    
    ./create_channel.py
    Usage: create_channel.py [options] --label <channel label> --name <channel name>
    
    Options:
      -h, --help           show this help message and exit
      --user=USER          username
      --password=PASSWORD  password
      --host=HOST          FQDN of your server
      --label=LABEL        channel label
      --name=NAME          channel name
      --summary=SUMMARY    channel summary
      --debug              enable debug output
    None


The second way to create a new channel is to use your Spacewalk website.

At the top of your Spacewalk page within the navigation tabs you will see *channels*.

![Alt](images/channel1.png?raw=True)

On the left hand navigation menu you will see *Manage Channels*.

![Alt](images/channel2.png?raw=True)

On the upper right hand corner of the body of the webpage you should see *Create New Channel*.

![Alt](images/channel3.png?raw=True)

Fill in the blanks accordingly and click *Create New*

![Alt](images/channel4.png?raw=True)

----
## Download the rpm packages



There are many ways to download content (rpms) [rsync](http://samba.anu.edu.au/rsync/), [wget](http://www.gnu.org/software/wget/), [reposync](http://wiki.linux.duke.edu/YumUtils), etc. Below we'll show two of these mechanisms: wget and reposync.

* *using wget*

One way of getting content is to use [wget](http://www.gnu.org/software/wget/) to fetch the packages from a given yum repo. While not the most elegant method, it works, using easily available tools. Let's get the Fedora 9 content and put them into a Fedora9 directory.


    # mkdir Fedora9
    # cd Fedora9
    # wget -r -l1 --no-parent \
            http://download.fedoraproject.org/pub/fedora/linux/releases/9/Everything/i386/os/Packages/

The same principal but for CentOS 5


    # mkdir CentOS5
    # cd CentOS5
    # wget -r -l1 --no-parent \
            http://mirror.linux.duke.edu/pub/centos/5/os/x86_64/CentOS/

* *using reposync*

A more elegant method of getting packages for uploading, is using [reposync](http://wiki.linux.duke.edu/YumUtils). [YumUtils](http://wiki.linux.duke.edu/YumUtils) has a really useful tool called [reposync](http://wiki.linux.duke.edu/YumUtils/RepoSync) to synchronize packages to a local directory.

 - First, install yum-utils if it isn't already: 

    yum install yum-utils
 - Install the desired repo in this case, fedora.

    $ head /etc/yum.repos.d/fedora.repo
    [fedora]
    name=Fedora $releasever - $basearch
    baseurl=http://download.fedoraproject.org/pub/fedora/linux/releases/$releasever/Everything/$basearch/os/
    mirrorlist=http://mirrors.fedoraproject.org/mirrorlist?repo=fedora-$releasever&arch=$basearch
    enabled=1
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora file:///etc/pki/rpm-gpg/RPM-GPG-KEY
 - Now pass in the repoid to reposync:

    $ reposync --repoid=fedora
    [fedora: 1     of 7390  ] Downloading Fedora/915resolution-0.5.3-1.fc7.i386.rpm
      915resolution-0.5.3-1.fc7 100% |=========================|  16 kB    00:00
    [fedora: 2     of 7390  ] Downloading Fedora/AGReader-1.2-2.fc6.i386.rpm
      AGReader-1.2-2.fc6.i386.r 100% |=========================|  47 kB    00:00
    [fedora: 3     of 7390  ] Downloading Fedora/AllegroOGG-1.0.3-3.fc6.i386.rpm
      AllegroOGG-1.0.3-3.fc6.i3 100% |=========================|  16 kB    00:00
      ...
 - Once the above step is done packages should be in the *fedora* directory:

    $ ls -l fedora/Fedora/
      total 18936
      -rw-rw-r-- 1 root root   15960 2007-05-23 10:18 915resolution-0.5.3-1.fc7.i386.rpm
      -rw-rw-r-- 1 root root   48528 2007-05-23 10:18 AGReader-1.2-2.fc6.i386.rpm
      -rw-rw-r-- 1 root root   16687 2007-05-23 10:19 AllegroOGG-1.0.3-3.fc6.i386.rpm
      ...

----
## Upload the rpm packages




Finally, it's time to push the content into the server. Once you have the rpms, use [rhnpush](http://www.redhat.com/docs/manuals/RHNetwork/ref-guide/chan-mgmt/4.1.0/s1-custom-pkgs-upload-sat.html) to push them to the Spacewalk server.

For example, to upload the content downloaded using *reposync* do the following:

    rhnpush --channel=fedora-9-i386 --server=http://localhost/APP --dir=fedora

*usage* see also [rhnpush](http://www.redhat.com/docs/manuals/RHNetwork/ref-guide/chan-mgmt/4.1.0/s1-custom-pkgs-upload-sat.html)

    rhnpush --channel=<channel-label> --server=<hosturl>/APP --dir=<directory>

Note: Make sure _/var/satellite_ exists and has owner:group _apache_ before pushing.

Note: I found that the above failed on Fedora 9 Everything.  My work around was:


    find . -name "*rpm" | xargs rhnpush --channel=fedora-9-i386 --server=http://localhost/APP -v --tolerant -u spacewalk -p spacewalk
