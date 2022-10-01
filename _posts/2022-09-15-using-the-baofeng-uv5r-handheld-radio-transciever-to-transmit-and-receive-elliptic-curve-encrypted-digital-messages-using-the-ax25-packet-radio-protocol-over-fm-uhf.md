---
layout: post
title: "Using the Baofeng UV5R handheld radio transciever to transmit and receive elliptic-curve encrypted digital messages using the AX.25 packet radio protocol over FM UHF"
categories: [IT, ham]
image: 
excerpt: "A practical guide on the concept of exchanging encrypted digital messages over AX.25 and UHF radio."
---

Here are the steps for that proof of concept:

- 1. generate two ECC key-pairs (Alice and Bob)
- 2. encrypt some message from Alice to Bob
- 3. convert that encrypted message into a .wav file
- 4. transmit that .wav audio over FM UHF (digital-to-analog)
- 5. record that transmission at another station into a new .wav file (analog-to-digital)
- 6. convert the new recorded .wav into the encrypted message string
- 7. decrypt the message


## 1. generate two ECC key-pairs (Alice and Bob)

The following OpenSSL commands should create two key-pairs aloingside their shared secrets.

```bash
# Generate private key files
$ openssl ecparam -name secp521r1 -genkey -noout -out alice-private-key.pem
$ openssl ecparam -name secp521r1 -genkey -noout -out bob-private-key.pem

# Generate public key files
$ openssl ec -in alice-private-key.pem -pubout -out alice-public-key.pem
$ openssl ec -in bob-private-key.pem -pubout -out bob-public-key.pem

# Generate shared secret files
$ openssl pkeyutl -derive -inkey alice-private-key.pem -peerkey bob-public-key.pem -out alice-shared-secret.bin
$ openssl pkeyutl -derive -inkey bob-private-key.pem -peerkey alice-public-key.pem -out bob-shared-secret.bin
```

It will create the following files:
- `alice-private-key.pem`
- `alice-public-key.pem`
- `alice-shared-secret.bin`
- `bob-private-key.pem`
- `bob-public-key.pem`
- `bob-shared-secret.bin`

Those files are for encrypt and decrypt the message.

## 2. encrypt some message from Alice to Bob

Now on the same directory of the `alice-shared-secret.bin`, let's create a message we want do encrypt. I'm going to create a file named `secret-message.txt`.

`secret-message.txt`
```
Steely eyes of a silvery people
Walk behind me, with evil intent.
Up to the tower, I stand alone
Stripped of rank before a throne I am sent.
What befalls us in the heat of the night?
```

And let's encrypt it for Bob:

```bash
$ openssl enc -aes256 -base64 -k $(base64 -w 0 alice-shared-secret.bin) -pbkdf2 -e -in secret-message.txt -out secret-message-cipher.txt
```

That will create a `secret-message-cipher.txt` file with the encrypted message, mine looks like so:

```
U2FsdGVkX1+Qni560oEGuJZ+GO2PjxiSolWyTRjCHHUkYJQd1nb1NjVRPvhVEOk9
ZG1m9Azk7MBJ6HXNaO6BVdIjO4PsJfQbeNn+sXYpBpueOHM48mT6JXo5iRLN6KBm
eEmkVOZRae8WvMJuPRvdwycrxshP9DcIypJGE/g/rbfkupfywOZY2sxGvDMjgi9i
45xCzWrVapUKiaC0L9pa5RmsSFxMKLcWM7rcLpJC6Hyhxs3W+K+A7gXwHX86xvAs
gZMakbxelzF8BK9tLjYs/w==
```

## 3 convert that encrypted message into a .wav file

For that I'm going to use the AX.25 protocol. Back then a piece of hardware (the modem) was needed in order to to perform the digital-to-analog/analog-to-digital convertion of the data into AX.25 packets, but nowadays that can be done with software, in this case here I'm using direwolf.

**Note 1:** Using the package manager direwolf distribution (AUR) resulted in some funky stuff for me, if you experience issues along the way then I recommend you to compile direwolf (dev branch) by yourself, following the instructions that will be displayed when you command `make` on direwolf's root directory.

**Note 2:** If compilation fails complaining about the libgps version, then check [this comment](https://github.com/wb2osz/direwolf/issues/402#issuecomment-1169398506), it should do the trick.

direwolf uses TAPR TNC2 monitoring format to send messages. Roughly speaking:
```
SOURCE>DESTINATION:payload
```

More about the format here.

I'm going to use ALICE and BOB as source and destination, and the payload should be the content of the `secret-message-cipher.txt` file.

So the following command should instruct direwolf's gen_packets program to create a `message-to-broadcast.wav` file with the encrypted message encoded in it using the TNC2 monitoring format over the AX.25 packet radio protocol:

```bash
$ echo -n 'ALICE>BOB:'$(cat secret-message-cipher.txt) | gen_packets -o message-to-broadcast.wav -
```

That `message-to-broadcast.wav` file should now be safely transmited over the radio.

## 4. transmit that audio over FM UHF (digital-to-analog)

### 4.1 warning
Before going any further I must warn you that doing so might be ilegal on your country, do that at your own risk and responsability. Be sure to use the correct part of the spectrum and to use low power settings. Ideally you should have an amateur radio license license to transmit. Only do this for testing or for emergency purposes.

To avoid problems I'm personally choosing a [Family Radio Service (FRS)](https://en.wikipedia.org/wiki/Family_Radio_Service) channel, setting the transmit power on the UV5R to low (which is 1 watt) and only transmitting it once or twice.

Here in my region the FRS channels are being used by all kinds of "pirates", and that makes UHF alive I think.

### 4.2 intro

That can be done in many different ways, the way I'm doing is by using the (already classic) Baofeng UV5R.

[PICTURE THE RADIO]

It usually comes alongside its cheap headset.

[![UV5R headset]({{ site.baseurl }}/images/2022-09-15-using-the-baofeng-uv5r-handheld-radio-transciever-to-transmit-and-receive-elliptic-curve-encrypted-digital-messages-using-the-ax25-packet-radio-protocol-over-fm-uhf/uv5r-headset.jpg)]({{ site.baseurl }}/images/2022-09-15-using-the-baofeng-uv5r-handheld-radio-transciever-to-transmit-and-receive-elliptic-curve-encrypted-digital-messages-using-the-ax25-packet-radio-protocol-over-fm-uhf/uv5r-headset.jpg)

I'm going to modify that headset to be able to send the radio's speaker audio signal to the computer, so I can record things I hear in the radio using Audacity. And I also want to be able to transmit audio from the computer using the radio. A simple modification in this cable can achieve both.

### 4.3 headset cable modification

**RADIO->PC**: Simply replace the earpiece with a p2 male connector. It's a mono signal so you can either use only one side of a stereo p2 or a dedicated mono p2 male connector. With that you can already record the radio signal on the computer, just hook that p2 on the mic-in or line-in of your computer.

**PC->RADIO**: For being able to transmit from the computer to the radio you will need to open the PTT/mic small plastic case and locate the small microphone unit. Desolder the unit and replace it with another male p2 connector. It's also a mono signal, so it's the same deal of the RADIO->PC sable. Then you can hook that in the speaker/headphone output of your computer. Mind that for trasmitting you will still need to push the PTT button. The plastic PTT/mic case will get too tight to close with the new cable in, but a bit of electrical tape will do it.

[PICTURE THE MOD CABLE]

### 4.4 transmit

Use low PC volume for trasmitting.  
Use medium radio volume for receiving.

Once the modification is completed, connect the PC->RADIO, pick a UHF frequency and be ready to push PTT while playing the `message-to-broadcast.wav` file.

## 5. record that transmission at another station into a new .wav file (analog-to-digital)
