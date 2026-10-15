# Network File Transfer and SDN

Two Python networking projects exploring application-layer file transfer over TCP and software-defined networking with Mininet, Ryu, and OpenFlow 1.3.

| Academic context | Details |
| --- | --- |
| University | Xi'an Jiaotong-Liverpool University (XJTLU) |
| Course | [CAN201 — Introduction to Networking](https://modules.xjtlu.edu.cn/mod?mod_code=CAN201&psl_code=SEM1) |
| Project type | Group coursework, Parts I and II |
| Portfolio owner | Junyan He |
| Team | Binyu Li, Junyan He, Chenyu Du, Zengyi Mei, Jiafei Sun |

This repository presents the team's coursework as part of Junyan He's academic portfolio. The implementations are shared team work; this page does not attribute the entire project to an individual member.

## Project overview

| Part | Focus | Main components |
| --- | --- | --- |
| [Part I: File transfer](part-1/) | A TCP client and server using the coursework STEP protocol | Block-based uploads and downloads, resume support, integrity checks, expiring tokens, and transfer metrics |
| [Part II: Software-defined networking](part-2/) | A virtual network with one client, two servers, and an Open vSwitch | A learning-based forwarding controller and a TCP traffic redirection controller |

## Part I — TCP file transfer

The STEP client and server exchange length-prefixed JSON metadata and binary payloads over TCP. The implementation includes:

- Chunked file upload and download with MD5 checks for transfer integrity.
- Upload status queries and resumption of missing blocks.
- Threaded transfer operations, retry handling, and token renewal when a token expires.
- Optional gzip/zlib compression and versioned uploads.
- File deletion, progress bars, and transfer performance logs.

The default upload path shares one TCP connection across worker threads, with a lock around socket access. The server handles connections using threads.

### Run locally

Use Python 3. From the repository root, create an environment and install the two client dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install numpy tqdm
cd part-1
```

Start the server in one terminal:

```bash
python server.py --ip 127.0.0.1 --port 1379
```

In a second terminal, activate the same environment, enter `part-1`, and upload a local file:

```bash
python client.py --server_ip 127.0.0.1 --id demo-user --f /path/to/example.txt
```

Use the same identifier for subsequent operations. It is a local coursework identifier; it does not need to be a real student number.

```bash
# Query an uploaded file by its server-side key.
python client.py --op status --server_ip 127.0.0.1 --id demo-user --key example.txt

# Download the file.
python client.py --op download --server_ip 127.0.0.1 --id demo-user --key example.txt --f downloads/example.txt

# Delete the file from this local test server.
python client.py --op delete --server_ip 127.0.0.1 --id demo-user --key example.txt
```

`python client.py --op upload --help` lists the extended CLI options, including `--compress` and `--version-control`. The client connects to port `1379`, so keep the server on that port unless you also change the client configuration. Runtime files, logs, and local environments are excluded by `.gitignore`.

The authentication and MD5-based token logic are coursework mechanisms for a local experiment. They are not intended as production authentication or encrypted file transfer.

## Part II — SDN forwarding and redirection

The topology places three hosts on one Open vSwitch connected to a remote Ryu controller:

```text
                       Ryu controller
                        OpenFlow 1.3
                             |
                            s1
                      /      |      \
                  client  server1  server2
                10.0.1.5 10.0.1.2 10.0.1.3
```

- [`networkTopo.py`](part-2/networkTopo.py) builds the Mininet topology and exposes controller IP/port options.
- [`ryu_forward.py`](part-2/ryu_forward.py) learns MAC addresses and installs bidirectional ICMP and initial-SYN TCP flows with a five-second idle timeout.
- [`ryu_redirect.py`](part-2/ryu_redirect.py) redirects TCP connections from the client, originally addressed to `server1`, toward `server2`. It rewrites IP/MAC headers in both directions so responses retain the logical identity of `server1`.

### Run in a networking lab environment

Use a Linux environment with Mininet, Open vSwitch, and a Python environment compatible with Ryu. These system dependencies are separate from the Part I dependencies. Run one controller at a time.

Start the forwarding controller from the repository root:

```bash
ryu-manager --ofp-tcp-listen-port 6633 part-2/ryu_forward.py
```

In another terminal, start the topology:

```bash
sudo python3 part-2/networkTopo.py --cip 127.0.0.1 --cport 6633
```

In the Mininet CLI, enable the protocol version used by the controller, check host connectivity, and inspect the installed flows:

```text
mininet> sh ovs-vsctl set bridge s1 protocols=OpenFlow13
mininet> pingall
mininet> sh ovs-ofctl -O OpenFlow13 dump-flows s1
```

For the redirection experiment, stop the topology and controller, then start the alternative controller and recreate the topology using the same commands above:

```bash
ryu-manager --ofp-tcp-listen-port 6633 part-2/ryu_redirect.py
```

Run `pingall` first so the controller can learn the hosts' ports, then send TCP traffic from `client` to `10.0.1.2`. The redirection controller installs forward and reverse rules for `server2`. Flow-table inspection or a packet capture can be used to observe the rewritten traffic. Use `exit` to stop the topology; `sudo mn -c` cleans up Mininet after an interrupted run.

## Repository structure

```text
.
├── README.md
├── .gitignore
├── part-1/
│   ├── client.py
│   └── server.py
└── part-2/
    ├── networkTopo.py
    ├── ryu_forward.py
    └── ryu_redirect.py
```

The repository contains the five Python source files supplied for the two coursework parts. The supplied `Report` folders contained no reports, so no report or assessment results are included here.

## Credits

**Team members:** Binyu Li, Junyan He, Chenyu Du, Zengyi Mei, and Jiafei Sun.

The source files are retained from the supplied group coursework, with the project organized into two descriptive directories and documentation added for navigation and local reproduction.
