# Setup

MuseBox Server is provided as an executable file.
There are 3 executable files:

- MuseBoxVideo (for video tasks)
- MuseBoxAudio (for audio tasks)
- MuseBoxVideoWithIP (for video tasks both DPU and custom IPs)


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

The server will be ready in about 30 seconds. When the server is online, it will print this information:
```
********************************************
* MuseBox Server is running at port 9696!  *
********************************************
```

# Pre-built image
You can ask us for the pre-built image by sending an email on `staff@makarenalabs.com`.

You only need to flash the image on a SD card with common image burner. We suggest BalenaEtcher as SD image burner.


# License request

In order to obtain the license, you need to request the node locker license.

**The license is valid for a specific board, so you cannot use the license on a different node.**

For the license request, you need to create the license request file. To do that, simply run this command:

`license_request`

This command generates the file `license_request.req` in the path `/usr/local` . You need to send to us the file via email at `staff@makarenalabs.com` with the subject `MuseBox License request` and with the message body `I accept your evaluation license agreement`.
Our internal system will check if the email is associated to a valid customer, then, according to your signed contract, the system will respond to you with the license file, called `license.lic` .
When you receive the license file, you need to place the file in this path:
`/usr/local/bin`

# Troubleshooting

If the machine learning tasks seems to give wrong responses, please restart your board.

---

If you need more help, contact us at:

https://discord.gg/NpkTaJPAdp

staff@makarenalabs.com
