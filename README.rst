SD card flasher Overview
=========================

The SD card flasher, also known as a gang programmer, is specially
designed and developed to simultaneously write to multiple SD cards and
make them bootable.

Features
---------

1. Automatic SD Card Detection: This feature enables the SD card flasher
to automatically recognize when an SD card is inserted into the
device. It eliminates the need for manual intervention, allowing
for a more streamlined and efficient operation. Once an SD card is
detected, the flasher can proceed with the necessary operations
without requiring user input.

2. Auto FAT32 Formatting: The SD card flasher is equipped with the
capability to automatically format SD cards to the FAT32 file
system. FAT32 is a commonly used file system for SD cards and
other removable storage devices. By automatically formatting the
SD cards to FAT32, the flasher ensures compatibility with a wide
range of devices and operating systems, simplifying the setup
process for users.

3. Runtime Memory Block SHAsum Verification: This functionality involves
verifying the integrity of the data written to the SD cards during
runtime using SHAsum (Secure Hash Algorithm). SHAsum is a
cryptographic hash function that generates a unique hash value for
a given set of data. By performing SHAsum verification on memory
blocks at runtime, the SD card flasher can ensure that the data
being written to the SD cards is accurate and has not been
corrupted during the flashing process. This verification adds an
extra layer of reliability and security to the flashing process,
reducing the risk of errors or data loss.

4. In the event of success or failure during any of these processes, the
flasher generates a comprehensive report detailing the outcome.
This report provides valuable feedback to the user, allowing them
to troubleshoot any issues that may arise and ensuring efficient
operation of the SD card flasher.


Tools used
-----------
   - ``dcfldd`` is an enhanced version of the dd command, commonly used for data
    copying and conversion tasks in Unix-like operating systems. 
    It offers features such as on-the-fly hashing, progress indicators, and 
    improved error handling compared to dd. This tool is particularly useful for tasks 
    like creating disk images, cloning disks, performing data recovery, and 
    verifying data integrity during copying operations. `document <https://linux.die.net/man/1/dcfldd>`__

Installation Guide for dcfldd
==============================

Introduction
------------

This guide provides step-by-step instructions on how to install `dcfldd`, an enhanced version of the `dd` command, on Unix-like operating systems.

Prerequisites
-------------

Before you begin, ensure that you have:
- Administrative privileges on the system.
- Access to the internet to download the necessary packages.

Installation Steps
------------------

1. **Update Package Repository**:
   - Open a terminal or command prompt.
   - Run the following command to update the package repository:

   .. code-block:: shell

      sudo apt update

2. **Install dcfldd**:
   - Once the repository is updated, run the following command to install `dcfldd`:

   .. code-block:: shell

      sudo apt install dcfldd

3. **Verification**:
   - After installation, you can verify that `dcfldd` is installed correctly by running the following command:

   .. code-block:: shell

      dcfldd --version

   - This command should display the version information for `dcfldd`, confirming that it's installed successfully.

Conclusion
----------

Congratulations! You have successfully installed `dcfldd` on your system. You can now use it for various data copying and conversion tasks.

.. note::

   - For systems other than Debian-based distributions, you may need to use a different package manager (e.g., `yum`, `dnf`) to install `dcfldd`.
   - Always ensure that you're downloading and installing software from trusted sources to avoid security risks.



Block Diagram
--------------

|

.. graphviz::

   digraph block_diagram {
       rankdir=LR;
       node [shape=rectangle];

       RaspberryPi [label="Raspberry Pi 4B"];
       SD1 [label="SD Card Reader 1"];
       SD2 [label="SD Card Reader 2"];
       PowerAdapter [label="Power Adapter"];

       RaspberryPi -> SD1 [label="USB", dir="both"];
       RaspberryPi -> SD2 [label="USB", dir="both"];
       PowerAdapter -> RaspberryPi [label="DC Power"];

       {rank=same; SD1; SD2;}
   }

.. .. image:: images/sd_flasher_block.png
..    :width: 3.44792in
..    :height: 2.6875in
| 

In addition to its functional capabilities, the flasher utilizes Linux
udev rules to detect USB devices. These udev rules enable the flasher to
recognize and interact with connected USB devices, including SD cards,
in a reliable and efficient manner within the Linux operating system
environment. This ensures seamless integration and compatibility with
Linux-based systems, enhancing the overall usability and versatility of
the flasher.

|
|

.. tab:: Unix/macOS

   .. code-block:: bash

      python3 -m pip --version

.. tab:: Windows

   .. code-block:: bat

      py -m pip --version


Program flow chart
-------------------

Here is the flowchart representation of the Bash script:

.. graphviz::

   digraph G {
       node [shape=rectangle, style="filled", fillcolor="#EAF2F8", fontname="Arial", fontsize=10, width=3, height=0.4];
       edge [fontname="Arial", fontsize=8];

       Start [label="Start",shape=oval];
       Reset [label="Reset Terminal"];
       Header [label="Display Header"];
       Hostname [label="Display Hostname"];
       CheckCards [label="Check SD Cards"];
       Format [label="Format SD Cards"];
       CheckFormat [label="Check Formatting Success"];
       WriteOS [label="Write OS Image"];
       Verify [label="Verify Image Write"];
       Success [label="Display Success"];
       Record [label="Record Results"];
       Error [label="Display Error and Retry"];
       End [label="End",shape=oval];

       Start -> Reset;
       Reset -> Header;
       Header -> Hostname;
       Hostname -> CheckCards;
       CheckCards -> Format [label="Yes"];
       CheckCards -> Error [label="No"];
       Format -> CheckFormat;
       CheckFormat -> WriteOS [label="Yes"];
       CheckFormat -> End [label="No"];
       WriteOS -> Verify;
       Verify -> Success [label="Yes"];
       Verify -> End [label="No"];
       Success -> Record -> End;
       Error -> Reset;
   }

|

Bash code
----------

.. code-block:: bash

    #!/bin/bash

    #set -eux

    # clear
    tput reset
    Red='\033[0;31m'          # Red
    Green='\033[0;32m'        # Green
    Yellow='\033[0;33m'       # Yellow
    Purple='\033[0;35m'       # Purple
    Cyan='\033[0;36m'         # Cyan
    White='\033[0;37m'        # White
    NC='\033[0m'              # No Color

    echo -e "${Green}"
    figlet -c inditronics sdcard flasher 
    echo -e "${NC}"
    echo -e "${Yellow}"
    hostname
    echo -e "${NC}"
    while true; do
      #if_sha256sum="856b1083c7d904c1595b24388734152fa81a0d4601cfa80b91dbd5588f78fbf3"
      if_sha256sum="c2d63f3fa884cc6443f44c99cce1eda5a4975e49cb18bb3cbae5eab0a5aed5d7"
    ##  if_sha256sum="db14120dfc5bf8e49068700d25023294a46c49e6e99137b6b55aa82763ef9447"
    #  if_sha256sum="aba7a1ced11c395c7300559620c3980ce1863e97b643e8b5319f7d5b001e59d1"
      DAY=`date +%A`
      DATE=`date +%m-%d-%Y`
      TIME=`date +"%r"`

      a=(`lsblk -d -n -p| grep sd | awk 'NR==1 {print $1}'`) # Store device path in veriable
      b=(`lsblk -d -n -p| grep sd | awk 'NR==2 {print $1}'`) # Store device path in veriable

      udevadm info $a | grep ID_VENDOR_ID | awk {'print $2'} | cut -c 14- > card1_VID.txt # Check SDcard VID and store in file  
      udevadm info $b | grep ID_VENDOR_ID | awk {'print $2'} | cut -c 14- > card2_VID.txt # Check SDcard VID and store in file

      udevadm info $a | grep ID_MODEL_ID | awk {'print $2'} | cut -c 13- > card1_MODEL.txt # Check SDcard VID and store in file  
      udevadm info $b | grep ID_MODEL_ID | awk {'print $2'} | cut -c 13- > card2_MODEL.txt # Check SDcard VID and store in file

      sd1_VID=`cat card1_VID.txt` # Read VID from file and store in veriable
      sd2_VID=`cat card2_VID.txt` # Read VID from file and store in veriable

      sd1_MODEL=`cat card1_MODEL.txt` # Read VID from file and store in veriable
      sd2_MODEL=`cat card2_MODEL.txt` # Read VID from file and store in veriable

      rm card1_VID.txt card2_VID.txt card1_MODEL.txt card2_MODEL.txt

      echo -e "SDcard1_VID :${Cyan} $sd1_VID ${NC} | SDcard2_VID :${Cyan} $sd2_VID ${NC}"
      echo -e "SDcard1_MODEL :${Cyan} $sd1_MODEL ${NC} | SDcard2_MODEL :${Cyan} $sd2_MODEL ${NC}"

      sleep 1
      if [ "$sd1_VID" = "05e3" ] && [ "$sd2_VID" = "05e3" ]; then
        echo "both VID matched"
        if [ -n "$a" ] || [ -n "$b" ]; then
          if [ -z "$a" ] || [ -z "$b" ]; then
            echo "$a $b please connect 2nd card"
            sleep 1
          fi
        else
          echo -e "${Yellow}$b $b Both cards are not found${NC}"
        fi  

        if [ -n "$a" ] && [ -n "$b" ]; then
          echo -e "${Cyan}formating $a ${NC}"; sudo mkfs.vfat -F 32 -I $a
          if [ "$?" = "0" ]; then
            echo -e "${Green}format${NC}${Purple} $a ${NC}${Green}completed${NC}"
          else
            echo -e "${Red}Fail to format $a ${NC}"
            break
          fi

          echo -e "${Cyan}formating $b ${NC}"; sudo mkfs.vfat -F 32 -I $b
          if [ "$?" = "0" ]; then
            echo -e "${Green}format${NC}${Purple} $b ${NC}${Green}completed${NC}"
          else
            echo -e "${Red}Fail to format $b ${NC}"
            break
          fi
          #echo "{Cyan}formating $a${NC}"; sudo mkfs.vfat -F 32 -I $b; echo -e "${Green}format${NC}${Purple} $b ${NC}${Green}completed${NC}"

     #******************************************/ Start SDcard Writing /***********************************************************#
          lsblk
          echo -e "${Green}***/${NC}${Yellow}OS Writing start${NC}${Green}/***${NC}"
          sudo dcfldd if=/home/pi/DCFL/bm4prod01_0_1_3.img bs=4096 hash=sha256  sha256log=/home/pi/DCFL/logs/$sd1_MODEL.sha256 of=$a&
          sudo dcfldd if=/home/pi/DCFL/bm4prod01_0_1_3.img bs=4096 hash=sha256  sha256log=/home/pi/DCFL/logs/$sd2_MODEL.sha256 of=$b

          sd1_sha256=`cat /home/pi/DCFL/logs/$sd1_MODEL.sha256 | awk 'NR==2{print $3}'`
          sd2_sha256=`cat /home/pi/DCFL/logs/$sd2_MODEL.sha256 | awk 'NR==2{print $3}'`

          echo "card1_sha256 : $sd1_sha256"
          echo "card2_sha256 : $sd2_sha256"
          # echo "main : $if_sha256sum"

          if [ "$sd1_sha256" = "$if_sha256sum" ] && [ "$sd2_sha256" = "$if_sha256sum" ]; then
            echo


code explanation
------------------

Initialization and Setup:

The script begins with some setup, including resetting the terminal (tput reset) and defining color variables for text output.
Display Information:

It displays a banner using figlet to indicate the purpose of the script.
It prints the hostname of the system.
Main Loop:

The script enters a while true; do loop, meaning it will continue to execute indefinitely until it encounters a break statement.
Check SD Card VID and Model:

It retrieves the vendor ID (VID) and model ID of two SD cards connected to the system.
The VID and model information is stored in variables (sd1_VID, sd2_VID, sd1_MODEL, sd2_MODEL) after processing.
Display SD Card Information:

It prints out the vendor IDs and model names of the two SD cards.
Check for Matching VID:

It checks if both SD cards have a specific vendor ID ("05e3").
Check Card Presence:

It verifies if both SD card paths ($a and $b) are not empty, indicating the presence of both cards.
If one or both cards are missing, it prompts to connect the missing card(s).
Format SD Cards:

If both cards are present, it formats them using mkfs.vfat with FAT32 filesystem and the -I option for quick formatting.
It checks if the formatting is successful and prints the appropriate message.
SD Card Writing:

It uses ``dcfldd`` to write an operating system image (bm4prod01_0_1_3.img) to both SD cards concurrently.
The output hash of each write operation is logged with SHA-256 checksum.
After writing, it retrieves the SHA-256 checksum from the log files.
Verify Write Operation:

It compares the obtained SHA-256 checksums with an expected checksum (if_sha256sum).
If both SD cards have the expected checksum, it indicates successful writing and logs the result in a CSV file.
If the checksums don't match, it displays a failure message.
End of Loop:
The loop breaks after completing the writing and verification process.
Failure Handling:
If the vendor IDs don't match, it prints a message indicating the mismatch and restarts the script (./dcfl.sh).
This script essentially automates the process of formatting and flashing OS images onto two SD cards while ensuring the integrity of the write operations through checksum verification.


.. _sdflash-guidelines:

sdflash Command Guidelines
=========================

Overview
--------

The ``sdflash`` command is designed to make two SD cards bootable simultaneously.
It ensures that both SD cards have the necessary operating system image written to them effectively.

Prerequisites
-------------

- Ensure that you have two SD cards connected to your system.
- Make sure the SD cards are properly inserted and recognized by the system.

Running the Command
-------------------

To run the ``sdflash`` command, open your terminal or command prompt, and execute the following:

.. code-block:: shell

    sdflash

Script Execution
----------------

Upon executing the ``sdflash`` command, the script will initialize and display relevant information.
It will check the Vendor ID (VID) of both connected SD cards to ensure compatibility.

Formatting SD Cards
-------------------

If the Vendor IDs match, the script will proceed to format both SD cards.
Formatting ensures that the SD cards are prepared for writing the operating system image.

Writing Operating System Image
-------------------------------

After formatting, the script will simultaneously write the operating system image to both SD cards.
This step involves transferring the necessary files to make the SD cards bootable.

Verification
------------

Once the writing process is complete, the script will calculate the SHA256 hash of each SD card's content.
It will compare the calculated hash with an expected value to ensure accurate data transfer.

Completion
----------

If the verification is successful for both SD cards, the script will display a success message.
It will log the results, indicating that the process was completed successfully.

Failure Handling
----------------

If any step in the process fails, the script will display an error message and prompt for necessary actions.
Error messages will guide you on how to troubleshoot and rectify the issue.

End of Execution
----------------

Once the script completes its execution, you can remove the SD cards from your system.
The SD cards are now bootable and ready for use in your desired applications.

.. note::


   - This command requires administrative privileges to execute certain operations like formatting and writing to the SD cards.
   - Ensure that you have the necessary permissions before running the ``sdflash`` command.
   - Exercise caution when working with storage devices to prevent data loss or damage.




.. |image1|
.. --------

.. .. |image1| image:: images/sd_flasher_flow.png
..    :width: 3.05729in
..    :height: 5.65924in
