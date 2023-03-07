# Setup

MuseBox Server is provided as an executable file.
There are 2 executable files:
- MuseBox Video (for video tasks)
- MuseBox Audio (for audio tasks)
  
The executable file are in `~/makarenalabs/musebox` called `MuseBoxVideo` and `MuseBoxAudio`. 


# Pre-built image
You can download the pre-built image from here: ...

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
