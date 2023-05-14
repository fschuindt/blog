---
layout: post
title: "Encrypted FM radio: Using the Baofeng UV5R handheld transceiver to transmit and receive elliptic-curve encrypted messages with the AX.25 packet radio protocol over UHF (Modemless)"
categories: [IT, ham]
image: images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/cover.jpg
excerpt: "A practical guide on the concept of exchanging encrypted digital messages over AX.25 and UHF with the Baofeng UV-5R radio."
---

![RX Cable]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/cover.jpg)

## Introduction

This text showcases a PoC on transmitting [Secp256k1](https://en.bitcoin.it/wiki/Secp256k1) elliptic-cuve encrypted digital text messages over frequency-modulated voice radio within the UHF spectrum, only for educational purposes. For modern and/or practical applications on the same band, please check [LoRa](https://www.semtech.com/lora/what-is-lora), [FSK](https://www.techtarget.com/searchnetworking/definition/frequency-shift-keying) and [ASK](https://www.elprocus.com/amplitude-shift-keying-ask-working-and-applications/).

It also offers an alternative (modemless) approach to the one displayed at the Computerphile's video [Packet Radio (Post Apocalyptic Internet?)](https://www.youtube.com/watch?v=lx6cm1rNDLM).

Here are the steps:

1. Generate two ECC key-pairs (Alice and Bob).
2. Encrypt a digital text message from Alice to Bob.
3. Convert the encrypted message into a `.wav` file.
4. Transmit the `.wav` audio over FM UHF (Digital-to-Analog).
5. Record the transmission at another station into a new `.wav` file (Analog-to-Digital).
6. Convert the new recorded `.wav` back into the encrypted message string.
7. Decrypt the message.

*RX Station*

[![RX Station]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/rx_station.jpg)]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/rx_station.jpg)

## 1. Generate two ECC key-pairs (Alice and Bob)

The following OpenSSL commands should create two key-pairs alongside their shared secrets.

```bash
# Generate private key files
openssl ecparam -name secp256k1 \
  -genkey -noout -out alice-private-key.pem

openssl ecparam -name secp256k1 \
  -genkey -noout -out bob-private-key.pem

# Generate public key files
openssl ec -in alice-private-key.pem \
  -pubout -out alice-public-key.pem

openssl ec -in bob-private-key.pem \
  -pubout -out bob-public-key.pem

# Generate shared secret files
openssl pkeyutl -derive \
  -inkey alice-private-key.pem \
  -peerkey bob-public-key.pem \
  -out alice-shared-secret.bin

openssl pkeyutl -derive \
  -inkey bob-private-key.pem \
  -peerkey alice-public-key.pem \
  -out bob-shared-secret.bin
```

It should create the following files:
```
alice-private-key.pem
alice-public-key.pem
alice-shared-secret.bin

bob-private-key.pem
bob-public-key.pem
bob-shared-secret.bin
```

## 2. Encrypt a digital text message from Alice to Bob

Now, on the same directory, let's write the message we want to encrypt. I'm going to set it as `secret-message.txt`.

`secret-message.txt`
```
New car, caviar, four star, daydream.
Think I'll buy me a football team.
```

And to encrypt it:

```bash
openssl enc -aes256 -base64 -k \
  $(base64 -w 0 alice-shared-secret.bin) \
  -pbkdf2 -e -in secret-message.txt \
  -out secret-message-cipher.txt
```

It will result in a `secret-message-cipher.txt` file, with the encrypted message.

Mine looks like so:
```
U2FsdGVkX1+mhD4zEq13II1kO7OBmPw7UuPWnedqSSC+9jJ/aXCFm6kF8bEtyBa0
4WUOk0Bawux+jKvQmSRRKvWNaRwdlTaWsZJyx9lWEqbubVY3FFWDCJ6L5A3K3kg6
```

## 3. Convert the encrypted message into a `.wav` file

For that, we're going to use the [AX.25 protocol](http://lea.hamradio.si/~s53mv/nbp/nbp/AX25V20.pdf) (See also [ax25.net](https://www.ax25.net/)).

In the past, a physical device (the modem) was needed in order to convert the digital signal into analog AX.25 packets (and vice versa). However, today we can find software-based modem implementations like [Dire Wolf](https://github.com/wb2osz/direwolf). Those allow us to perform RF AX.25 communications without any physical devices other than the radio units and their respective cables.

To send messages, Dire Wolf uses the [TAPR TNC2's monitoring message format](http://www.aprs-is.net/Specification.aspx) (See also 1. [This comment](https://github.com/n2ygk/aprs-bigdata/issues/1#issuecomment-462080468) 2. [The KISS protocol](https://www.ax25.net/kiss.aspx)). Simplifying for our needs, we can use the following format adaptation:
```
SOURCE>DESTINATION:payload
```

While `SOURCE` and `DESTINATION` are reserved for stations call signs, for the purposes of this PoC we're going to use `ALICE` and `BOB`. The payload should be, in this case, the content of the `secret-message-cipher.txt` file.

Example:
```
ALICE>BOB:U2FsdGVkX1+mhD4zEq13II1kO7OBmPw [...]
```

The following command should instruct Dire Wolf's `gen_packets` program to create a `message-to-broadcast.wav` file with the encrypted message modulated as AX.25 packets:

```bash
echo -n 'ALICE>BOB:'$(cat secret-message-cipher.txt) \
  | gen_packets -o message-to-broadcast.wav -
```

That file should now be transmitted over the radio. Fun fact: If you listen to it, it sounds just like a Dial-Up connection.

`message-to-broadcast.wav` file:
<audio controls>
  <source src="{{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/message-to-broadcast.wav" type="audio/wav">
</audio>

## 4. Transmit the `.wav` audio over FM UHF (Digital-to-Analog)

I'm going to use 1W TX power at the 462.587MHz (UHF) frequency, which is [FRS](https://www.radioreference.com/db/aid/7732)' Channel 2. And of course, the Baofeng uses [narrow FM](https://www.rcet.org.in/uploads/academics/rohini_985418e52915.pdf).

AFAIK, regulations-wise, AX.25 packets can not be sent within the FRS bands. However, I'm going to do it just for the sake of this demonstration. Moreover, here in my region all these channels are being used by all kinds of *pirates*, including digital. In *some sense*, I think this is good, as it maintains the hobby alive. And we all expect a lot of noise within UHF anyway. But here goes a formal disclaimer.

### 4.1. Warning
Before going any further, I must warn you that doing so might be illegal on your country. Please consult local regulations. Be sure to use the correct part of the spectrum and to use the correct power settings. Ideally, you should have a local license for transmitting.

With that, we continue.

### 4.2. Cable modification

Every Baofeng UV-5R comes with a PTT, microphone and speaker cable, just like this one:


<img src="{{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/uv5r-headset.jpg" style="max-width: 60%;" />

I modified two of these cables, one to act as the TX cable and the other to act as the RX cable. However, if you are interested into full-duplex capabilities, you should modify your cables using the TX schematics (follows), which will support both TX and DX.

### 4.2.1 TX Cable

[![TX Cable]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/finished_tx_cable_annotated.jpg)]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/finished_tx_cable_annotated.jpg)

Schematics ([Open Source project](https://oshwlab.com/f.schuindtcs/baofeng-uv5r-audio-cables)):
[![Schematics]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/schematics.png)]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/schematics.png)

And I replaced both the microphone and the speaker with a p2-stereo male cable each. Even though the signal is mono, those were the pieces I had lying around, as most of this was made with recycled parts.

I also replaced the PTT button for a slightly bigger one.

### 4.2.2 RX Cable

[![RX Cable]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/finished_rx_cable_annotated.jpg)]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/finished_rx_cable_annotated.jpg)

Schematics:
[![RX Cable Drawing]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/rx_cable_draw.jpg)]({{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/rx_cable_draw.jpg)

### 4.4 Transmit

Here follows a simple diagram of what's happening. ([Open here](https://gist.github.com/fschuindt/e8d94b0c8802931eeca3c99ad857cde5) for rendering it)

<script src="https://gist.github.com/fschuindt/e8d94b0c8802931eeca3c99ad857cde5.js"></script>

The TX cable is connecting one UV-5R's microphone to ALICE's computer p2 audio output. The RX cable connects the other UV-5R's speaker to BOB's computer p2 line input. With the two radios turned on and tuned to the same frequency, BOB's computer starts to record the audio signal from the line input. Then on ALICE, the PTT is pressed, and the `message-to-broadcast.wav` is played, which will be transmitted by the radio, then recorded by BOB's computer.

## 5. Record the transmission at another station into a new `.wav` file (Analog-to-Digital)

On BOB's computer, the recording results in a `recorded-broadcast.wav` file:
<audio controls>
  <source src="{{ site.baseurl }}/images/2023-05-14-encrypted-fm-radio-using-the-baofeng-uv5r-handheld-transceiver-to-transmit-and-receive-elliptic-curve-encrypted-messages-with-the-ax25-packet-radio-protocol-over-uhf-modemless/recorded-broadcast.wav" type="audio/wav">
</audio>

## 6. Convert the new recorded `.wav` back into the encrypted message string

Now we can use Dire Wolf's `atest` program to demodulate it back into text:
```
$ atest recorded-broadcast.wav

44100 samples per second.  16 bits per sample.  2 audio channels.
1314816 audio bytes in file.  Duration = 7.5 seconds.
Fix Bits level = 0
Channel 0: 1200 baud, AFSK 1200 & 2200 Hz, E, 44100 sample rate.
Channel 1: 1200 baud, AFSK 1200 & 2200 Hz, E, 44100 sample rate.

DECODED[1] 0:03.587 ALICE audio level = 0(0/0)
[0] ALICE>BOB:U2FsdGVkX1+mhD4zEq13II1kO7OBmPw7UuPWnedqSSC+9jJ/aXCFm6kF8bEtyBa0 4WUOk0Bawux+jKvQmSRRKvWNaRwdlTaWsZJyx9lWEqbubVY3FFWDCJ6L5A3K3kg6


1 from v1.wav
1 packets decoded in 0.093 seconds.  80.6 x realtime
```

And sure enough, the text is there:
```
[0] ALICE>BOB:U2FsdGVkX1+mhD4zEq13II1kO7OBmPw7UuPWnedqSSC+9jJ/aXCFm6kF8bEtyBa0 4WUOk0Bawux+jKvQmSRRKvWNaRwdlTaWsZJyx9lWEqbubVY3FFWDCJ6L5A3K3kg6
```

## 7. Decrypt the message.

`atest` added a white space instead of a `\n`, but this is no problem:

```
U2FsdGVkX1+mhD4zEq13II1kO7OBmPw7UuPWnedqSSC+9jJ/aXCFm6kF8bEtyBa0
4WUOk0Bawux+jKvQmSRRKvWNaRwdlTaWsZJyx9lWEqbubVY3FFWDCJ6L5A3K3kg6
```

That's the exact same text. Now, just decrypt it:

```bash
openssl enc -aes256 -base64 \
  -k $(base64 -w 0 bob-shared-secret.bin) \
  -pbkdf2 -d -in extracted-message.txt \
  -out result.txt \
  && cat result.txt
```

Which outputs:
```
New car, caviar, four star, daydream.
Think I'll buy me a football team.
```

Great!

<div style="height: 40px;">
</div>

+ Cover picture: *Midjourney prompt: illuminated manuscript, medieval art, pre-renaissance, a radio operator using a computer --v 5 --aspect 7:4 --seed 722*
+ Thanks to the YouTube user "Karst & Coffee" for [mentioning](https://www.youtube.com/watch?v=lx6cm1rNDLM&lc=Ugx27BsF9xXMnlQD3Gx4AaABAg.8zZ02hXCMrW8zb-Sq6zcYN) about software modems.
