# CEC Sniffer Libray
This module will connect to a kwikwai device using its web api.

kwikwai constanyl writes new blocks in order and thus we can query
the upcoming block, whicever is the new block we consider it to be new.

This module works in an async and a sync way, yes kinda like node js :)

##class CecSniffer:
Parameters

hostname : string
    the URL of the kwikwai device, aboid ntebios
    and bonjour  on unix

Attributes

current_last_message : string
    the current buffer we query (new block appears here)
messages : obj
    messages inside each block
token : int
    random int from kiwkwai, used as auth token
hostname

##__init__(self, hostname)
Parameters

hostname : string
    the URL of the kwikwai device, avoid ntebios
    and bonjour  on unix

Returns

none
    constructs :o

##begin_listen(self,cec_bus_listener,cec_bus_listener_keepalive,cec_sniffer_killed,cec_sniffer_started):

Parameters

cec_bus_listener : callback
    for ASYNC, will be called on new message
cec_bus_listener_keepalive : callback
    for ASYNC will be called to avoid test forever
cec_sniffer_killed : callback
    for AYNC, to be called when listener killed gracefully
cec_sniffer_started : callback
    for ASYNC, to be called on new listener

Returns

class
    the constructed class

##sync_messages_local(self)

This is the secret sauce, this should be called on each
module start as this will check all the previous blocks
of data in the kwikwai buffer and import them into a
local variable. this is so we can keep checking for
new blocks and use our local blocks to compare any new ones
that appear in kwikwais memory
Returns

json
no return

##send_message(self, message)

sends a message through a POST.
we also sniff the token to include
in this post
  Parameters

  message : string
      the message to seng
  Returns

  true
      return true, for non blocking

##wait_for_next(self)
link for the non persistant_query
Returns

block (json)
will return the messsage

##wait_for_next_rx(self, message)
link for the non persistant_query but will
also send a message. will raise rx flag
Parameters

message : json
message to send to kwikwai

Returns

json
Description of returned object.

##beta_interpreter(self, block)
will attempt to convert a recieved message into json
Parameters

block : json
block of recieved data from kwikwai

Returns

json
obj with interpreted messages from kwikwai,
format is set below

#Tests (Sample)

##Async Sample
```
def CEC_bus_listener_keepalive(last_block_index):
    print("~ Quering CEC bus for blocks after -> " + str(last_block_index))

def CEC_bus_listener(block):
    print(block)
    CEC_block_interpreter(block)

def CEC_sniffer_killed():
        print("~ Trying blocking argument, synchronous waiting for next...")
        print(sniffer.wait_for_next())

def CEC_sniffer_started():
    print("~ Persistant Listener Started")

def CEC_block_interpreter(block):
    for message in block["messages"]:
        readeable_hex = []
        for block in message["blocks"]:
            if(len(str(hex(int(block))))<6):
                readeable_hex.append("00")
            else:
                readeable_hex.append(str(hex(int(block)))[2:-2])
        print("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~")
        print("Message (human readeable not correct)->
        " + ":".join(readeable_hex))
        sent_or_not = "TX"
        if(message["sent"]==False):
            sent_or_not = "RX"
        print("RX/TX                                -> " + sent_or_not)
        print("Bus                                  -> " + message["bus"])

print("~ Syncing previous sniffed blocks...wait...")
sniffer = CEC_Sniffer("http://172.20.44.138")
#no NetBIOS or Bonjour on linux :(
sniffer.sync_messages_local()
sniffer.begin_listen(CEC_bus_listener,
CEC_bus_listener_keepalive,
CEC_sniffer_killed, CEC_sniffer_started)
```

## Sync/Blocking Example
```
def set_up_interface(hostname):
    try:
        sniffer = CEC_Sniffer(hostname) #no NetBIOS or Bonjour on linux :(
        logger.debug("[cec-sniffer] Percussively inspecting CEC bus buffer...")
        sniffer.sync_messages_local()
        logger.debug("[cec-sniffer] Buffer synced!")
        return sniffer
    except:
        fail("CEC Interface Error")

cec_listener = set_up_interface(hostname)
for message in cec_listener.wait_for_next_rx("2F:85:20:00"):
    message_parsed = cec_listener.beta_interpreter(message)
    print(message_parsed)
```
## Sync Sample No RX Expectance
```
def set_up_interface(hostname):
    try:
        sniffer = CEC_Sniffer(hostname) #no NetBIOS or Bonjour on linux :(
        logger.debug("[cec-sniffer] Percussively inspecting CEC bus buffer...")
        sniffer.sync_messages_local()
        logger.debug("[cec-sniffer] Buffer synced!")
        return sniffer
    except:
        fail("CEC Interface Error")
cec_listener.send_message("2F:85:20:00")
cec_listener = set_up_interface(hostname)
for message in cec_listener.wait_for_next():
    message_parsed = cec_listener.beta_interpreter(message)
    print(message_parsed)

```

## Full Test Implementation Sample

```
from tvos.cec_sniffer import CecSniffer
#Must include
import logging
import matplotlib
matplotlib.use('agg')
import matplotlib.pyplot as plt
import common
from testcases import attr
from utils.testing import fail, given, then, when

@attr(trid="C497220")

def set_up_interface(hostname):
    try:
        sniffer = CecSniffer(hostname) #no NetBIOS or Bonjour on linux :(
        logger.debug("[cec-sniffer] Percussively inspecting CEC bus buffer...")
        sniffer.sync_messages_local()
        logger.debug("[cec-sniffer] Buffer synced!")
        return sniffer
    except:
        fail("CEC Interface Error")

def cec_test(hostname="http://172.20.44.138"):
    cec_listener = set_up_interface(hostname)
    with given(""" The STB is connected through HDMI to a device that displays and decodes CEC messages exchanged between devices on the HDMI bus:
        Ref: https://wiki.youview.co.uk/display/canvas/Using+the+Kwikwai+device+to+inspect+HDMI+CEC+messages
        Note: Please ensure HDMI 1 on the TV is used during setup.

        AND the STB is the HDMI Active source
        To check that STB hdmi-cec-state is "Active", send an HDMI CEC <Request Active Source> message on the CEC device e.g 2F:85:20:00 and the STB should respond by broadcasting an HDMI CEC <Active Source> message 3F:82:10:00 """):


        sent_message = False
        recieved_message = False
        for message in cec_listener.wait_for_next_rx("2F:85:20:00"):
            message_parsed = cec_listener.beta_interpreter(message)
            print(message_parsed)
            try:
                if message_parsed["direction"] == "TX" and message_parsed["message"] == "2f:85:20:00":
                    sent_message = True

                if message_parsed["direction"] == "RX" and message_parsed["message"].split(":")[1] == "82":
                    recieved_message = True
            except Exception as e:
                fail('Error decoding messages')
        preconditions_output = {
            "sent_message" : sent_message,
            "recieved_message" : recieved_message
        }
        print(preconditions_output)
        if not (sent_message and recieved_message):
            #raise common.InvalidTest('Could not check STB is the HDMI active source, probably because we have a splitter inline')
            logger.debug('Could not check STB is the HDMI active source, probably because we have a splitter inline')
            #fail("Could not check STB is the HDMI active source")

    with when(""" I send an HDMI CEC "Request Active Source" message on the CEC device: """):
        print("Testing beta feature")

    with then("""  The STB will broadcast an HDMI CEC "Active Source (82)" message """):
        sent_message = False
        recieved_message = False
        for message in cec_listener.wait_for_next_rx("4F:85:30:00"):
            message_parsed = cec_listener.beta_interpreter(message)
            print(message_parsed)
            try:
                if message_parsed["direction"] == "TX" and message_parsed["message"] == "4f:85:30:00":
                    sent_message = True

                if message_parsed["direction"] == "RX" and message_parsed["message"].split(":")[1] == "82": #We can simply check the message ID from the 2nd block and return the correct request "Active Source" (Brodcast).
                    recieved_message = True
            except:
                fail("Failed on awaiting Active Source (82), check combined log")
        if not (sent_message and recieved_message):
            preconditions_output = {
                "sent_message" : sent_message,
                "recieved_message" : recieved_message
            }
            logger.debug(preconditions_output)
            fail("Failed on awaiting Active Source (82), check combined log")

```
