---
layout: post
title: Wireless network fundamentals
description: Introduce wireless network fundamentals
location: Taiwan
tags:
- Network
- Wireless
---

## Wireless network fundamentals
Wireless networks uses radio signals (electromagnetic waves) to transmit data over the air withoutns relying any physical connection, enable the desired convenience and mobility for users.
Provide the following advantages over traditional wired networks:
- Mobility: wireless network users can connect to existing networks, allowing an increase in productivity and effectiveness of operations.
- Flexibility: Wireless networks are capable of adapting network configuration easily.
- Reduce cost of ownership: overall expenses can be significantly lower than wired network hardware.

### Types of wireless networks
There're four types of wireless networks: Wireless PAN, Wireless LAN, Wireless MAN and Wireless WAN.

- `Wireless PAN`
Wireless personal area network covers a relatively small area that is generally 100 meters for most applications, supported protocols like [Bluetooth](https://en.wikipedia.org/wiki/Bluetooth) and [Zigbee](https://en.wikipedia.org/wiki/Zigbee)
- `Wireless LAN`
Wireless local area network is based on high frequency radio waves allow establishing two or more devices over a short distance. A WLAN connection has several applications such as in organizations and industries to provide a connection through an access point for Internet access.

- `Wireless MAN`
Wireless metropolitan area network connects several wireless LANs to provide a wider area than office or home networks. There're two types of Wireless MAN
  - `Back haul`
  It's an enterprise type of network (e.g. cellular) that connects with the global internet and other core network access locations. A DSL can be used for back haul but wireless conenction si 10 times faster than normal optical fiber connection.
  - `Last mile`
  Last mile connection is used to network for a temporary period, it's used to make a network in some construction buildings and is an alternative to DSL broadband and cable modem with a low cost and quick installation.
- `Wireless WAN`
Wireless wide area network use cellular technology to provide access outside the range of a wireless LAN or metropolitan network.

| Type | Range | Applications | Standards |
|------|-------|--------------|-----------|
| Personal area network (PAN) | Within reach of a person | Cable replacement for peripherals | Bluetooth, ZigBee, NFC |
| Local area network (LAN) | Within a building or campus | Wireless extension of wired network | IEEE 802.11 (WiFi) |
| Metropolitan area network (MAN) | Within a city | Wireless inter-network connectivity | IEEE 802.15 (WiMAX) |
| Wide area network (WAN) | Worldwide | Wireless network access | Cellular (UMTS, LTE, etc) |

(Learn more about [browser networking](https://hpbn.co/introduction-to-wireless-networks/#types-of-wireless-networks))

### Performance fundamentals of wireless networks
An application of the channel capacity concept to additive white Gaussian noise (AWGN) channel with bandwidth and signal-to-noise ratio theorem
[Additive white Gaussian noise](https://wikimedia.org/api/rest_v1/media/math/render/svg/7544c459b09cdddfb1e86553bb340d7067868bc1)
- C is the channel capacity and is measured in bits per second.
- B is the available bandwidth and is measured in hertz.
- S is signal and N is noise and they are measured in watts.

> It's also worth noting that not all frequency ranges offer the same performance.
Low frequency signals travel farther and cover large areas, but at the cost of requiring larger antennas and having more clients competing for access.
High frequency signals can transfer more data but won't travel as far, resulting in smaller coverage areas and a requirement for more infrastructure.

A few factors that may affect the performance of wireless network:
- Amount of distance between receiver and sender
- Amount of background noise in current location
- Amount of interference from users in the same network (inter-cell)
- Amount of interference from users in other, nearby networks (inter-cell)
- Amount of available transmit power, both at receiver and sender
- Amount of processing power and the chosen modulation scheme

To maximize the throughput of wireless network, try to remove any noise and interference, place receiver and sender as close as possible and give them all the power they desire and make sure both select the best modulation method.
