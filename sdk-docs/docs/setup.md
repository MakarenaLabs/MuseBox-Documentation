# Setup

MuseBox Server is provided as an executable file.

If you will use the pre-built image, the executable file is in `~/makarenalabs/musebox-demo` called `MuseBox`. 

If you will install MuseBox on your petalinux image, the executable file is in `/usr/local/bin`, called `MuseBoxServer`.

# Pre-built image
If you want to use the pre-built image, you only need to flash the image on a SD card with common image burner. We suggest BalenaEtcher as SD image burner.


# Installation on your Petalinux image
If you want to install MuseBox on top of your current petalinux image, follow those steps

## Prerequisites
 - Petalinux 2021.1 image
 - PYNQ v2.7 (following steps)
 - Valid license provided by MakarenaLabs SRL

## Step 1: PYNQ installation

To install PYNQ you can simply use the provided RPM package with the follwing command:
```bash
dnf install pynq-2.7-1.fc30.noarch.rpm
```

After that, to apply all changes **you have to reboot the board and wait a couple of seconds before start using PYNQ on Kria board**. Jupyter notebooks are available on `9090` port, so you can reach them using the URL `http://board_ip:9090`.

## Step 2: MuseBox server installation

To start using MuseBox Server you have to install it by an RPM package with the following command (it takes about 20 mins):
```bash
dnf install musebox_server-2021.1_1.4-1.fc30.noarch.rpm
```

## Configuration

First of all, you need to check if the file `dpu.xclbin` is visible to the system, so check if the file is in the path 	`/media/sd-mmcblk0p1/`. If it is not true, you have to link the DPU file (`dpu.xclbin`) into a specific directory inside `/media` folder. You can do this with the following:
```bash
mkdir -p /media/sd-mmcblk0p1/
ln -s /run/media/mmcblk1p1/dpu.xclbin /media/sd-mmcblk0p1/
```
At your discretion, you can also copy the file in the directory, instead of linking it (you have to change the `ln -s` command with the `cp` one).

Once the DPU file is located in the right directory, you can copy your `license.lic` file inside the MuseBox Server installation directory (`/usr/local/bin`).

# License request

In order to obtain the license, you need to request the node locker license.

**The license is valid for a specific board, so you cannot use the license on a different node.**

For the license request, you need to create the license request file. To do that, simply run this command:

`license_request`

This command generates the file `license_request.req` in the path `/usr/local` . You need to send to us the file via email at `staff@makarenalabs.com` with the subject `MuseBox License request` .
Our internal system will check if the email is associated to a valid customer, then, according to your signed contract, the system will respond to you with the license file, called `license.lic` .
When you receive the license file, you need to place the file in this path:
`/usr/local/bin`
