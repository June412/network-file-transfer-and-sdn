# Network File Transfer and SDN

**CAN201 — Introduction to Networking**

Two Python networking projects: [TCP file transfer](part-1/) with resumable uploads and integrity checks, and [software-defined networking](part-2/) with OpenFlow packet forwarding and TCP traffic redirection.

## Run

**File transfer:** Requires Python 3, `numpy`, and `tqdm`. From the repository root, run the server and client in separate terminals:

```bash
python3 part-1/server.py --ip 127.0.0.1
python3 part-1/client.py --server_ip 127.0.0.1 --id demo-user --f /path/to/file.txt
```

**SDN:** Requires Linux, Mininet, Open vSwitch, and Ryu. Run these in separate terminals:

```bash
ryu-manager --ofp-tcp-listen-port 6633 part-2/ryu_forward.py
sudo python3 part-2/networkTopo.py
```

Then in the Mininet prompt:

```text
sh ovs-vsctl set bridge s1 protocols=OpenFlow13
pingall
```

Use `ryu_redirect.py` instead of `ryu_forward.py` to explore TCP traffic redirection.
