# BGP/MPLS VPN Visualizer

An interactive, step-by-step animation of how a BGP/MPLS IP VPN works, covering both the **control plane** (how routes and labels are learned) and the **forwarding plane** (how a customer packet crosses the provider backbone).

It follows the model in [RFC 4364](https://www.rfc-editor.org/rfc/rfc4364) (which obsoletes RFC 2547).

**Live demo:** https://sunilgama.github.io/bgp-mpls-vpn-visualizer/

## What it shows

One customer with two sites, connected through a provider MPLS backbone.

```
Host A - CE1 - PE1 - P1 - P2 - PE2 - CE2 - Host B
 site A                                      site B
10.1.0.0/24        provider AS 65000         10.2.0.0/24
```

### Control plane (7 steps)

1. The IGP (OSPF or IS-IS) advertises the PE loopbacks.
2. LDP binds transport labels to those loopbacks, with penultimate hop popping (PHP).
3. PE1 and PE2 open an MP-iBGP session for the VPNv4 address family.
4. PE2 learns the site B prefix from CE2 into VRF RED.
5. PE2 exports it as a VPNv4 route: route distinguisher, route target, VPN label and next hop.
6. PE1 imports the route into VRF RED by matching the route target.
7. PE1 advertises the prefix to CE1 as a plain IPv4 route.

### Forwarding plane (7 steps)

A packet travels from Host A (10.1.0.10) to Host B (10.2.0.10):

1. Host A sends plain IP.
2. CE1 forwards it to PE1 on an interface bound to VRF RED.
3. PE1 pushes two labels: VPN label 100, then transport label 20 on top.
4. P1 swaps the top label (20 to 30). It never looks at the VPN label or the IP header.
5. P2 pops the transport label (PHP).
6. PE2 pops the VPN label and looks up the destination in VRF RED.
7. Plain IP is delivered to Host B.

The label stack is drawn on the packet at every hop, and each step shows the relevant table entries (VRF, LFIB, BGP UPDATE contents).

## How to use

- Switch between **Control plane** and **Forwarding plane** with the tabs.
- Use **Back**, **Next** or **Play**, and change the speed with the selector.
- Click any step in the list to jump to it.
- Keyboard: left and right arrow keys move between steps.

Colour key: amber is the transport label (LDP), teal is the VPN label (BGP), violet is a control-plane message, grey is the customer IP packet.

## Run locally

No build step or dependencies. Download `index.html` and open it in a browser.

The page loads the IBM Plex fonts from Google Fonts. If you are offline it falls back to system fonts and works the same.

## Assumptions

- One customer and one VRF (RED).
- eBGP between CE and PE.
- LDP for transport labels, with PHP enabled.
- Full-mesh iBGP between two PEs, no route reflector.
- All addresses, label values, RD (65000:100) and RT (65000:100) are example values chosen for this walkthrough.

## Possible extensions

- A second VRF with overlapping addresses, to show why RDs are needed.
- A route reflector between the PEs.
- RSVP-TE or Segment Routing as the transport instead of LDP.
- Return-path animation from Host B to Host A.

## References

- [RFC 4364](https://www.rfc-editor.org/rfc/rfc4364): BGP/MPLS IP Virtual Private Networks (VPNs)
- [RFC 2547](https://www.rfc-editor.org/rfc/rfc2547): original BGP/MPLS VPNs (obsoleted by RFC 4364)
- [RFC 3031](https://www.rfc-editor.org/rfc/rfc3031): MPLS architecture
- [RFC 3032](https://www.rfc-editor.org/rfc/rfc3032): MPLS label stack encoding
- [RFC 4271](https://www.rfc-editor.org/rfc/rfc4271): BGP-4
- [RFC 4360](https://www.rfc-editor.org/rfc/rfc4360): BGP extended communities
- [RFC 4760](https://www.rfc-editor.org/rfc/rfc4760): Multiprotocol extensions for BGP-4
- [RFC 5036](https://www.rfc-editor.org/rfc/rfc5036): LDP specification

## License

MIT. See the `LICENSE` file.
