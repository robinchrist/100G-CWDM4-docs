This repository aims to collect some information about the very popular 100GBase-CWDM4 transceivers. Those transceivers are cheap, versatile (most can do 100G and 40G), and SMF (Single Mode Fiber) based which makes them very cheap to use.

The [programmings](programmings) directory contains the binary transceiver programmings as well as their decoded form (using my [TransceiverTool](https://github.com/robinchrist/TransceiverTool)) as text.
The [datasheets](datasheets) directory contains some datasheets for the transceivers.

# List of 100G-CWDM4 transceivers
| Name | Reprogrammable | Distance |
|------|----------------|----------|
|Finisar FTLC1152RGPL6-FB|Yes, with password 0x00 0x00 0x10 0x11|500m|
|Finisar FTLC1157RGPL6FB1|Yes, with password 0x00 0x00 0x10 0x11|500m|
|Innolight TR-FC13H-HFB|**No** (Default password from other Inolight does not work)|500m|
|Intel SPTSBP2CLCKS|**No** (Passowrd unknown)|500m|
|Kaiam XQX5170|**No** (Passowrd unknown)|2km (from programming, no datasheet available)
|Innolight TR-FC13T-NAZ|Yes, with password 0x00 0x00 0x10 0x11|2km|

## Transceiver Specific Notes
### Finisar FTLC1152RGPL6-FB
In Revelprog-IS, go to `password`, select `enter current password`, then enter `00 00 10 11`, click `excute` and then **DESELECT Enter current password`
For some reason reprogramming will not work if `Enter current password` stays selected.
Once unlocked, the transceiver will stay unlocked forever, even after power cycle!

The programming indicates 2km maximum distance, but this is false. The transceivers is a `100G-CWDM4 Lite` and therefore 500m.
The datasheet indicates that the `6` in FTLC1152RGPL**6** means `Lite reach (500m) and limited temperature range`

## Finisar FTLC1157RGPL6FB1
In Revelprog-IS, go to `password`, select `enter current password`, then enter `00 00 10 11`, click `excute` and then **DESELECT Enter current password`
For some reason reprogramming will not work if `Enter current password` stays selected.
Once unlocked, the transceiver will stay unlocked forever, even after power cycle!

The programming indicates 2km maximum distance, but this is false. The transceivers is a `100G-CWDM4 Lite` and therefore 500m.
The datasheet indicates that the `6` in FTLC1152RGPL**6** means `Lite reach (500m) and limited temperature range`

## Innolight TR-FC13H-HFB
Surprisingly, this one does not use the expected QSFP default password `0x00 0x00 0x10 0x11`.
Current password unknown, or not reprogrammable at all


# Running 100GBase-CWDM4 at 40G with Mellanox NICs

**TL;DR: If you want to run 100GBase-CWDM4 transceivers at 40G with Mellanox NICs ConnectX-4 and newer (ConnectX-4, ConnectX-5, ConnectX-6) the transceiver MUST be programmed with the Mellanox OUI `00:02:c9`, otherwise no link will be established (the card actively disables the laser!)**

It can be desirable to run 100GBase-CWDM4 transceivers at 40GBit/s, e.g. to connect a server to a switch, as 100GBase-CWDM4 transceivers are often cheaper and better available than 40GBase-LR (or 40GBase-LR Lite) and allow an upgrade path to 100Gbit without buying new transceivers, making them a much better investment.

Sadly, this is not trivial if you have Mellanox NICs, specifically ConnectX-4 and newer (ConnectX-4, ConnectX-5, ConnectX-6).

As an example, we will use a Mellanox CX416A-CCAT (ConnectX-4 4x100G QSFP28) and Finisar FTLC1152RGPL6-FB transceivers.
Let's force the ports to 40Gbit/s

```
ethtool -s ens16f1np1 speed 40000 duplex full autoneg off
ethtool -s ens16f0np0 speed 40000 duplex full autoneg off
```
(note that `autoneg off` or `autoneg on` does not make a difference).

We will be greeted with

```
ethtool ens16f1np1

Settings for ens16f1np1:
[...]
        Advertised link modes:  40000baseKR4/Full
                                40000baseCR4/Full
                                40000baseSR4/Full
                                40000baseLR4/Full
[...]
        Link detected: no (Cable issue, Unsupported cable)
```

<details>
  <summary>Full ethtool ens16f1np1</summary>
  
```
ethtool ens16f1np1
Settings for ens16f1np1:
      Supported ports: [ FIBRE ]
      Supported link modes:   1000baseKX/Full
                              10000baseKR/Full
                              40000baseKR4/Full
                              40000baseCR4/Full
                              40000baseSR4/Full
                              40000baseLR4/Full
                              25000baseCR/Full
                              25000baseKR/Full
                              25000baseSR/Full
                              50000baseCR2/Full
                              50000baseKR2/Full
                              100000baseKR4/Full
                              100000baseSR4/Full
                              100000baseCR4/Full
                              100000baseLR4_ER4/Full
      Supported pause frame use: Symmetric
      Supports auto-negotiation: Yes
      Supported FEC modes: None        RS      BASER
      Advertised link modes:  40000baseKR4/Full
                              40000baseCR4/Full
                              40000baseSR4/Full
                              40000baseLR4/Full
      Advertised pause frame use: No
      Advertised auto-negotiation: No
      Advertised FEC modes: None       RS      BASER
      Speed: Unknown!
      Duplex: Unknown! (255)
      Auto-negotiation: off
      Port: FIBRE
      PHYAD: 0
      Transceiver: internal
      Supports Wake-on: d
      Wake-on: d
      Link detected: no (Cable issue, Unsupported cable)

```
  
</details>

Oh no! No link.
If we look at the `ethtool -m ens16f1np1` output we will see that the Mellanox card does actively disables the laser!
```
ethtool -m ens16f1np1

[...]
Transmit avg optical power (Channel 1)    : 0.0001 mW / -40.00 dBm
Transmit avg optical power (Channel 2)    : 0.0001 mW / -40.00 dBm
Transmit avg optical power (Channel 3)    : 0.0001 mW / -40.00 dBm
Transmit avg optical power (Channel 4)    : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 1)  : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 2)  : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 3)  : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 4)  : 0.0001 mW / -40.00 dBm
[...]
```

Solution: We need to reprogram the transceivers.

Right now they are looking like this:
```
ethtool -m ens16f1np1
[...]
Vendor name                               : FINISAR CORP.
Vendor OUI                                : 00:90:65
Vendor PN                                 : FTLC1152RGPL6-FB
[...]
```

<details>
  <summary>Full ethtool -m ens16f1np1</summary>
  
```
ethtool -m ens16f1np1

Identifier                                : 0x11 (QSFP28)
Extended identifier                       : 0xcc
Extended identifier description           : 3.5W max. Power consumption
Extended identifier description           : CDR present in TX, CDR present in RX
Extended identifier description           : High Power Class (> 3.5 W) not enabled
Power set                                 : Off
Power override                            : On
Connector                                 : 0x07 (LC)
Transceiver codes                         : 0x80 0x00 0x00 0x00 0x00 0x00 0x00 0x00
Transceiver type                          : 100G Ethernet: 100G CWDM4 MSA with FEC
Encoding                                  : 0x07 ((256B/257B (transcoded FEC-enabled data))
BR, Nominal                               : 25500Mbps
Rate identifier                           : 0x02
Length (SMF,km)                           : 2km
Length (OM3 50um)                         : 0m
Length (OM2 50um)                         : 0m
Length (OM1 62.5um)                       : 0m
Length (Copper or Active cable)           : 0m
Transmitter technology                    : 0x40 (1310 nm DFB)
Laser wavelength                          : 1301.000nm
Laser wavelength tolerance                : 6.500nm
Vendor name                               : FINISAR CORP.
Vendor OUI                                : 00:90:65
Vendor PN                                 : FTLC1152RGPL6-FB
Vendor rev                                : A0
Vendor SN                                 : UX503M0
Date code                                 : 170131
Revision Compliance                       : SFF-8636 Rev 2.5/2.6/2.7
Module temperature                        : 34.50 degrees C / 94.10 degrees F
Module voltage                            : 3.2706 V
Alarm/warning flags implemented           : Yes
Laser tx bias current (Channel 1)         : 0.000 mA
Laser tx bias current (Channel 2)         : 0.000 mA
Laser tx bias current (Channel 3)         : 0.000 mA
Laser tx bias current (Channel 4)         : 0.000 mA
Transmit avg optical power (Channel 1)    : 0.0001 mW / -40.00 dBm
Transmit avg optical power (Channel 2)    : 0.0001 mW / -40.00 dBm
Transmit avg optical power (Channel 3)    : 0.0001 mW / -40.00 dBm
Transmit avg optical power (Channel 4)    : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 1)  : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 2)  : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 3)  : 0.0001 mW / -40.00 dBm
Rcvr signal avg optical power(Channel 4)  : 0.0001 mW / -40.00 dBm
Laser bias current high alarm   (Chan 1)  : Off
Laser bias current low alarm    (Chan 1)  : On
Laser bias current high warning (Chan 1)  : Off
Laser bias current low warning  (Chan 1)  : On
Laser bias current high alarm   (Chan 2)  : Off
Laser bias current low alarm    (Chan 2)  : On
Laser bias current high warning (Chan 2)  : Off
Laser bias current low warning  (Chan 2)  : On
Laser bias current high alarm   (Chan 3)  : Off
Laser bias current low alarm    (Chan 3)  : On
Laser bias current high warning (Chan 3)  : Off
Laser bias current low warning  (Chan 3)  : On
Laser bias current high alarm   (Chan 4)  : Off
Laser bias current low alarm    (Chan 4)  : On
Laser bias current high warning (Chan 4)  : Off
Laser bias current low warning  (Chan 4)  : On
Module temperature high alarm             : Off
Module temperature low alarm              : Off
Module temperature high warning           : Off
Module temperature low warning            : Off
Module voltage high alarm                 : Off
Module voltage low alarm                  : Off
Module voltage high warning               : Off
Module voltage low warning                : Off
Laser tx power high alarm   (Channel 1)   : Off
Laser tx power low alarm    (Channel 1)   : On
Laser tx power high warning (Channel 1)   : Off
Laser tx power low warning  (Channel 1)   : On
Laser tx power high alarm   (Channel 2)   : Off
Laser tx power low alarm    (Channel 2)   : On
Laser tx power high warning (Channel 2)   : Off
Laser tx power low warning  (Channel 2)   : On
Laser tx power high alarm   (Channel 3)   : Off
Laser tx power low alarm    (Channel 3)   : On
Laser tx power high warning (Channel 3)   : Off
Laser tx power low warning  (Channel 3)   : On
Laser tx power high alarm   (Channel 4)   : On
Laser tx power low alarm    (Channel 4)   : On
Laser tx power high warning (Channel 4)   : On
Laser tx power low warning  (Channel 4)   : On
Laser rx power high alarm   (Channel 1)   : Off
Laser rx power low alarm    (Channel 1)   : On
Laser rx power high warning (Channel 1)   : Off
Laser rx power low warning  (Channel 1)   : On
Laser rx power high alarm   (Channel 2)   : Off
Laser rx power low alarm    (Channel 2)   : On
Laser rx power high warning (Channel 2)   : Off
Laser rx power low warning  (Channel 2)   : On
Laser rx power high alarm   (Channel 3)   : Off
Laser rx power low alarm    (Channel 3)   : On
Laser rx power high warning (Channel 3)   : Off
Laser rx power low warning  (Channel 3)   : On
Laser rx power high alarm   (Channel 4)   : Off
Laser rx power low alarm    (Channel 4)   : On
Laser rx power high warning (Channel 4)   : Off
Laser rx power low warning  (Channel 4)   : On
Laser bias current high alarm threshold   : 55.000 mA
Laser bias current low alarm threshold    : 25.000 mA
Laser bias current high warning threshold : 50.000 mA
Laser bias current low warning threshold  : 30.000 mA
Laser output power high alarm threshold   : 3.5481 mW / 5.50 dBm
Laser output power low alarm threshold    : 0.1585 mW / -8.00 dBm
Laser output power high warning threshold : 1.7783 mW / 2.50 dBm
Laser output power low warning threshold  : 0.3981 mW / -4.00 dBm
Module temperature high alarm threshold   : 60.00 degrees C / 140.00 degrees F
Module temperature low alarm threshold    : 10.00 degrees C / 50.00 degrees F
Module temperature high warning threshold : 55.00 degrees C / 131.00 degrees F
Module temperature low warning threshold  : 15.00 degrees C / 59.00 degrees F
Module voltage high alarm threshold       : 3.6300 V
Module voltage low alarm threshold        : 2.9700 V
Module voltage high warning threshold     : 3.4650 V
Module voltage low warning threshold      : 3.1350 V
Laser rx power high alarm threshold       : 2.2387 mW / 3.50 dBm
Laser rx power low alarm threshold        : 0.0251 mW / -16.00 dBm
Laser rx power high warning threshold     : 1.7783 mW / 2.50 dBm
Laser rx power low warning threshold      : 0.0631 mW / -12.00 dBm
```
  
</details>

The Vendor OUI `00:90:65` corresponds to `00:90:65 Finisar Corporation`

Let's change this to `00:02:C9 Mellanox Technologies, Inc.`. After reprogramming:

```
ethtool -m ens16f1np1

[...]
Vendor name                               : FINISAR CORP.
Vendor OUI                                : 00:02:c9
Vendor PN                                 : FTLC1152RGPL6-FB
[...]
```
We have changed the Vendor OUI from `00:90:65 Finisar Corporation` to `00:02:C9 Mellanox Technologies, Inc.`, nothing more! (Okay, and of course the checksum...)

Let's check out link again:

```
ethtool ens16f1np1

Settings for ens16f1np1:
[...]
        Advertised link modes:  40000baseKR4/Full
                                40000baseCR4/Full
                                40000baseSR4/Full
                                40000baseLR4/Full
[...]
        Speed: 40000Mb/s
        Link detected: no (Cable issue, Unsupported cable)
```

Hooray! We've got a 40Gbit link over 100GBase-CWDM4 transceivers.

For the sake of completeness, full output after reprogramming:

<details>
  <summary>Full ethtool -m ens16f1np1</summary>
  
```
ethtool -m ens16f1np1

Identifier                                : 0x11 (QSFP28)
Extended identifier                       : 0xcc
Extended identifier description           : 3.5W max. Power consumption
Extended identifier description           : CDR present in TX, CDR present in RX
Extended identifier description           : High Power Class (> 3.5 W) not enabled
Power set                                 : Off
Power override                            : On
Connector                                 : 0x07 (LC)
Transceiver codes                         : 0x80 0x00 0x00 0x00 0x00 0x00 0x00 0x00
Transceiver type                          : 100G Ethernet: 100G CWDM4 MSA with FEC
Encoding                                  : 0x07 ((256B/257B (transcoded FEC-enabled data))
BR, Nominal                               : 25500Mbps
Rate identifier                           : 0x02
Length (SMF,km)                           : 2km
Length (OM3 50um)                         : 0m
Length (OM2 50um)                         : 0m
Length (OM1 62.5um)                       : 0m
Length (Copper or Active cable)           : 0m
Transmitter technology                    : 0x40 (1310 nm DFB)
Laser wavelength                          : 1301.000nm
Laser wavelength tolerance                : 6.500nm
Vendor name                               : FINISAR CORP.
Vendor OUI                                : 00:02:c9
Vendor PN                                 : FTLC1152RGPL6-FB
Vendor rev                                : A0
Vendor SN                                 : UX503M0
Date code                                 : 170131
Revision Compliance                       : SFF-8636 Rev 2.5/2.6/2.7
Module temperature                        : 33.76 degrees C / 92.77 degrees F
Module voltage                            : 3.2505 V
Alarm/warning flags implemented           : Yes
Laser tx bias current (Channel 1)         : 42.068 mA
Laser tx bias current (Channel 2)         : 37.926 mA
Laser tx bias current (Channel 3)         : 38.870 mA
Laser tx bias current (Channel 4)         : 37.968 mA
Transmit avg optical power (Channel 1)    : 1.0778 mW / 0.33 dBm
Transmit avg optical power (Channel 2)    : 1.0394 mW / 0.17 dBm
Transmit avg optical power (Channel 3)    : 0.9198 mW / -0.36 dBm
Transmit avg optical power (Channel 4)    : 1.1533 mW / 0.62 dBm
Rcvr signal avg optical power(Channel 1)  : 0.9969 mW / -0.01 dBm
Rcvr signal avg optical power(Channel 2)  : 0.9100 mW / -0.41 dBm
Rcvr signal avg optical power(Channel 3)  : 0.9101 mW / -0.41 dBm
Rcvr signal avg optical power(Channel 4)  : 0.9713 mW / -0.13 dBm
Laser bias current high alarm   (Chan 1)  : Off
Laser bias current low alarm    (Chan 1)  : Off
Laser bias current high warning (Chan 1)  : Off
Laser bias current low warning  (Chan 1)  : Off
Laser bias current high alarm   (Chan 2)  : Off
Laser bias current low alarm    (Chan 2)  : Off
Laser bias current high warning (Chan 2)  : Off
Laser bias current low warning  (Chan 2)  : Off
Laser bias current high alarm   (Chan 3)  : Off
Laser bias current low alarm    (Chan 3)  : Off
Laser bias current high warning (Chan 3)  : Off
Laser bias current low warning  (Chan 3)  : Off
Laser bias current high alarm   (Chan 4)  : Off
Laser bias current low alarm    (Chan 4)  : Off
Laser bias current high warning (Chan 4)  : Off
Laser bias current low warning  (Chan 4)  : Off
Module temperature high alarm             : Off
Module temperature low alarm              : Off
Module temperature high warning           : Off
Module temperature low warning            : Off
Module voltage high alarm                 : Off
Module voltage low alarm                  : Off
Module voltage high warning               : Off
Module voltage low warning                : Off
Laser tx power high alarm   (Channel 1)   : Off
Laser tx power low alarm    (Channel 1)   : Off
Laser tx power high warning (Channel 1)   : Off
Laser tx power low warning  (Channel 1)   : Off
Laser tx power high alarm   (Channel 2)   : Off
Laser tx power low alarm    (Channel 2)   : Off
Laser tx power high warning (Channel 2)   : Off
Laser tx power low warning  (Channel 2)   : Off
Laser tx power high alarm   (Channel 3)   : Off
Laser tx power low alarm    (Channel 3)   : Off
Laser tx power high warning (Channel 3)   : Off
Laser tx power low warning  (Channel 3)   : Off
Laser tx power high alarm   (Channel 4)   : Off
Laser tx power low alarm    (Channel 4)   : Off
Laser tx power high warning (Channel 4)   : Off
Laser tx power low warning  (Channel 4)   : Off
Laser rx power high alarm   (Channel 1)   : Off
Laser rx power low alarm    (Channel 1)   : Off
Laser rx power high warning (Channel 1)   : Off
Laser rx power low warning  (Channel 1)   : Off
Laser rx power high alarm   (Channel 2)   : Off
Laser rx power low alarm    (Channel 2)   : Off
Laser rx power high warning (Channel 2)   : Off
Laser rx power low warning  (Channel 2)   : Off
Laser rx power high alarm   (Channel 3)   : Off
Laser rx power low alarm    (Channel 3)   : Off
Laser rx power high warning (Channel 3)   : Off
Laser rx power low warning  (Channel 3)   : Off
Laser rx power high alarm   (Channel 4)   : Off
Laser rx power low alarm    (Channel 4)   : Off
Laser rx power high warning (Channel 4)   : Off
Laser rx power low warning  (Channel 4)   : Off
Laser bias current high alarm threshold   : 55.000 mA
Laser bias current low alarm threshold    : 25.000 mA
Laser bias current high warning threshold : 50.000 mA
Laser bias current low warning threshold  : 30.000 mA
Laser output power high alarm threshold   : 3.5481 mW / 5.50 dBm
Laser output power low alarm threshold    : 0.1585 mW / -8.00 dBm
Laser output power high warning threshold : 1.7783 mW / 2.50 dBm
Laser output power low warning threshold  : 0.3981 mW / -4.00 dBm
Module temperature high alarm threshold   : 60.00 degrees C / 140.00 degrees F
Module temperature low alarm threshold    : 10.00 degrees C / 50.00 degrees F
Module temperature high warning threshold : 55.00 degrees C / 131.00 degrees F
Module temperature low warning threshold  : 15.00 degrees C / 59.00 degrees F
Module voltage high alarm threshold       : 3.6300 V
Module voltage low alarm threshold        : 2.9700 V
Module voltage high warning threshold     : 3.4650 V
Module voltage low warning threshold      : 3.1350 V
Laser rx power high alarm threshold       : 2.2387 mW / 3.50 dBm
Laser rx power low alarm threshold        : 0.0251 mW / -16.00 dBm
Laser rx power high warning threshold     : 1.7783 mW / 2.50 dBm
Laser rx power low warning threshold      : 0.0631 mW / -12.00 dBm

```
  
</details>

<details>
  <summary>Full ethtool ens16f1np1</summary>
  
```
ethtool ens16f1np1
Settings for ens16f1np1:
        Supported ports: [ FIBRE ]
        Supported link modes:   1000baseKX/Full
                                10000baseKR/Full
                                40000baseKR4/Full
                                40000baseCR4/Full
                                40000baseSR4/Full
                                40000baseLR4/Full
                                25000baseCR/Full
                                25000baseKR/Full
                                25000baseSR/Full
                                50000baseCR2/Full
                                50000baseKR2/Full
                                100000baseKR4/Full
                                100000baseSR4/Full
                                100000baseCR4/Full
                                100000baseLR4_ER4/Full
        Supported pause frame use: Symmetric
        Supports auto-negotiation: Yes
        Supported FEC modes: None        RS      BASER
        Advertised link modes:  40000baseKR4/Full
                                40000baseCR4/Full
                                40000baseSR4/Full
                                40000baseLR4/Full
        Advertised pause frame use: No
        Advertised auto-negotiation: No
        Advertised FEC modes: None       RS      BASER
        Speed: 40000Mb/s
        Duplex: Full
        Auto-negotiation: off
        Port: FIBRE
        PHYAD: 0
        Transceiver: internal
        Supports Wake-on: d
        Wake-on: d
        Link detected: yes
```
  
</details>