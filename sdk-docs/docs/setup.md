# Setup

MuseBox Server is provided as an executable file.
There are 2 executable files:
- MuseBoxVideo (for video tasks)
- MuseBoxVideoWithIP (for video tasks both DPU and custom IPs)
- MuseBoxAudio (for audio tasks)
  
The executable file are in `/usr/local/bin` called `MuseBoxVideo`, `MuseBoxVideoWithIP` and `MuseBoxAudio`. For starting a server, simply run the preferred MuseBox command.

For example, if you want to run MuseBox server for video task, run:

```
# for websocket communication
MuseBoxVideo websocket
```
```
# for zmq communication
MuseBoxVideo zmq
```



# Pre-built image
You can download the pre-built image from here: [DOWNLOAD NOW!](https://tinyurl.com/2lxb4gnn)

You only need to flash the image on a SD card with common image burner. We suggest BalenaEtcher as SD image burner.


# License request

In order to obtain the license, you need to request the node locker license.

**The license is valid for a specific board, so you cannot use the license on a different node.**

For the license request, you need to create the license request file. To do that, simply run this command:

`license_request`

This command generates the file `license_request.req` in the path `/usr/local` . You need to send to us the file via email at `staff@makarenalabs.com` with the subject `MuseBox License request` .
Our internal system will check if the email is associated to a valid customer, then, according to your signed contract, the system will respond to you with the license file, called `license.lic` .
When you receive the license file, you need to place the file in this path:
`/usr/local/bin`

# Troubleshooting

If you have this error when you run the MuseBox server or you request the license: 

`Unknown device "/dev/mmcblk0": No such file or directory` 

please launch this command: 

`ln -s /dev/mmcblk1 /dev/mmcblk0`


---

If the machine learning tasks seems to give wrong responses, please restart your board.

---

If you need more help, contact us at:

https://discord.gg/NpkTaJPAdp

staff@makarenalabs.com
