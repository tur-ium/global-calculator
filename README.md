global-calculator
=================

The source code to the web interface to the Global Calculator.

For more information about the global calculator, see: http://www.globalcalculator.org

To play with this web interface, go to: http://tool.globalcalculator.org

Canonical source: http://github.com/decc/global-calculator

Please report issues and suggest patches there.

Installation
-----------

This guide is adapted from Tom Counsell’s guide on the decc GitHub. You can find a more up to date fork of the project here.
## What you need
1. A computer with at least 4 GB and ideally 8 GB of memory 
2. Windows 10 or Linux
3. Internet access. You will need to download up to 3 Gb of files, though this can be reduced to less.
For Linux
4. Standard build tools (g++, etc) 
5. With version 2.1 2.5 of Ruby installed, including development headers 

In the util folder is a bash script that we use to set up Ubuntu 14.04 20.04 to be capable of running the global calculator. This can give clues on how to get the system running.

## Windows 10 Instructions (Skip if you have Linux)
### Setup the Windows Subsystem for Linux (WSL)
_Refer to these instructions if you have issues (where these instructions are adapted from): https://docs.microsoft.com/en-us/windows/wsl/install-win10_
1. Open PowerShell as an administrator by typing “PowerShell” into the search bar, and selecting “Run as administrator”. If prompted, press confirm or enter an administrator password.
2. In the blue Powershell window copy and paste (or type) the following command, all as one line:
   dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
3. Restart your machine to complete the WSL install and update to WSL 2. 
4. Open the Microsoft Store and select your favourite Linux distribution. This guide is designed for Ubuntu, but can be adapted to other flavours of Linux.

## Installing Ruby and Bundle
The following is adapted from these instructions for installing Ruby using RVM, tested on Ubuntu 20.04 LTS on Windows 10 WSL 1.0. Here we use Ruby 2.5, but if this doesn’t work try a newer version of Ruby.
    1. \curl -sSL https://get.rvm.io | bash
    2. rvm install 2.5
    3. rvm use 2.5
    4. gem install bundler
## Download and setup the Global Calculator
Run the following commands in Linux. To save time or if you have limited internet access, skip steps 6-8. This means you will not see images of possible future heat maps, but the server will run fine
    1. git clone http://github.com/decc/global-calculator 
    2. cd global-calculator 
    3. bundle update
    4. bundle 
    5. cd model/global_2050_model 
    6. bundle exec rake 
    7. cd ../../public/gc-anim 
    8. wget -nH --cut-dirs=1 --no-parent -r http://d2ow8032j7094s.cloudfront.net/20150123/index.html # This downloads 1.5 GB of images 
    9. cd ../.. 

## Starting the server

10. Go to the main global-calculator directory and run the following command rackup. Some lines of code will appear, and eventually something along the lines of WEBrick Server started … Port 9292
11.  Open a web browser, like Firefox and go to http://localhost:9292 . You should see the homepage of the global calculator – all running on your own computer!

## Notes
The bundle step should install all the dependencies. If it fails it may ask you to check a particular 'gem' installs manually. Doing that normally fixes the problem and the step can be repeated.
The bundle exec Rake step compiles the C version of the model. This can take tens of minutes, and requires plenty of memory.

Altering the spreadsheet
------------------------

1. Edit the spreadsheet in your spreadsheet software of choice
2. Transfer the file to the server
    1. Method 1 (preferred):
        1. Upload the file to `model/2050Model.xlsx` on your GitHub repository. 
        2. Go to your server and run ‘git pull origin master’
    2. Method 2 (if using Windows 10 and WSL) :
        1. Go to your server and type in `explorer.exe .` (including the dot after the space)
        2. You should now see a File Explorer window with your Linux files (which are in a virtual network directory called “\\wsl$\”)
        3. Copy and paste your spreadsheet to `model/2050Model.xlsx` in your Linux files
    3. In Linux, make sure you have unzip installed. You can do this by running ‘sudo apt install unzip’
    4. In your model folder on Linux, run `ruby translate_excel_into_c.rb`. This code (written by the great Tom Counsell) will compile your spreadsheet into c and ruby code that the webserver can use, and will take a long time, since it is a pretty complex spreadsheet. On a laptop with an i5 processor and 8 Gb RAM this took about 2 hours, but it could take less time, or more depending on your computer. You could get a server with some powerful processors, contribute to the excel_to_c code on GitHub to make it more efficient, run this overnight, or get a cup of tea to make this go more quickly.       
    5. Once this finishes running. Navigate to the `global_2050_model directory` by running ‘cd global_2050_model’
    6. Run ‘bundle exec rake’
    7. Navigate to the main directory by doing ‘cd ../..’
    8. Start the server to check it is working by running ‘rackup’
Note:

1. that the C version only includes outputs that are given in named ranges starting with 'webtool'
2. That translating a spreadsheet of this size needs at least 4 GB of RAM and can take 5 or 6 hours.


The gc-anim folder
------------------

The gc-anim folder contains several GB of climate change maps. These are too large to be placed under git version control.

Instead, we store a copy of the gc-anim folder in Amazon S3. The images can be accessed at http://d2ow8032j7094s.cloudfront.net/20150123/index.html

Requests to the EC2 instance for these images get redirected by nginx to the CloudFront cache, which in turn gets them from S3.

To get a local copy of these images for offline use, follow the instructions in public/gc-anim/README.md

Scaling
-------

The live version scales up and down using an autoscaler. This is set to watch for an alarm when the CPU load of the instances in the load balancer go above 20%, and then deploy an extra instance based on a launch configuration that uses an AMI that we have pre-prepared.

Updating
--------

Changes to the gc-anim folder need to be re-synced with Amazon S3 (see public/gc-anim/README.md).

If the code has come from Markus's version, common things that need to change include:

1. Removing references to ``_V22`` in ruby code, particularly in the model folder
2. Removing ``puts`` statements in ``hi-4-x.rb`` script
2. Changing the global_calculator_model require in the ``hi-4-x.rb`` ``require_relative '../model/global_2050_model'`` 

Changes should be tested locally, then committed to github.com

Then, deploy a new Ubuntu 14.04 instance, using the existing ami. Give that instance a public IP.

ssh into that instance, and do git pull to get the new code. 

Then restart nginx ``sudo service nginx restart``

Test whether that server seems to be providing the right result.

Then create an ami from that server.

Then create a new launch configuratio using that ami.

Then change the auto scale group to use that launch configuration.

Then kill off one of the existing running instances, and wait to see the new instance created. Check that it is using the right ami. Possibly assign it a public IP address so you can check it is serving the right things.

Then kill off any remaining running instances, and wait to see them recreated.

Check everything is working.

If not, switch the auto scale group back to the old launch configuration, then kill all the new instances.




