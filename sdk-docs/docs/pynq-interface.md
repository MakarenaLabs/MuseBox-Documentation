# PYNQ Interface

We are thrilled to announce that we have developed the Python bindings for our MuseBox AI product. Why is this beneficial? We can explore FPGA programming, machine learning, and multimedia processing in a simple and convenient way. And, we can build and customize our own AI pipeline and project using MuseBox in a much easier way.


## Setup

Firstly, for the setup, you will need a KV260 board, on which is running Ubuntu 22.04 LTS, PYNQ, PYNQ-DPU, and MuseBox.

Therefore, the first step to do is to install Ubuntu 22.04. We are using the latest version from [Install Ubuntu on Xilinx | Ubuntu](https://ubuntu.com/download/amd-xilinx).

Secondly, you should install [Kria-PYNQ](https://github.com/Xilinx/Kria-PYNQ) and [PYNQ-DPU](https://github.com/Xilinx/DPU-PYNQ). It may take some time.

Finally, once you finish installing everything we mentioned, MakarenaLabs provides a .so file that should be put on `/usr/local/share/pynq-venv/lib/python3.10/site-packages/`. You can request this file by sending an email to `staff@makarenalabs.com`.

You are now ready to start and create your own AI pipelines.

## License request

In order to obtain the license, you need to request the node locker license.

**The license is valid for a specific board, so you cannot use the license on a different node.**

For the license request, you need to create the license request file. To do that, simply run this command:

`license_request`

This command generates the file `license_request.req` in the path `/usr/local` . You need to send to us the file via email at `staff@makarenalabs.com` with the subject `MuseBox License request` and with the message body `I accept your evaluation license agreement`.
Our internal system will check if the email is associated to a valid customer, then, according to your signed contract, the system will respond to you with the license file, called `license.lic` .
When you receive the license file, you need to place the file in this path:
`/usr/local/bin`

## Usage

<video width="720"  controls>
  <source src="../assets/videos/musebox_pynq.mp4" type="video/mp4">
</video>



If you need more help, contact us at:

https://discord.gg/NpkTaJPAdp

staff@makarenalabs.com