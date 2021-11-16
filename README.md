# wifi-test-suite-orchestrator

A tiny script to automate Wifi tests, it needs to be used with the [Wi-Fi Test Suite Linux Control Agent](https://github.com/Wi-FiTestSuite/Wi-FiTestSuite-Linux-DUT)

# Setup

```sh
pip3 install -r requirements.txt
```

# Usage

```sh
./executor -f test_file.txt
```

# Test definition

Test are described in a human readable text format.

## Comments

Comments are prefixed with the `#` symbol

**Format**

```sh
# Comment
```

**Example**

```sh
# This is a comment
```

## Peers definition

All peers interacting during the test execution are declared using the following format:

**Format**

```csv
;peer,peer_name,peer_ip_address,peer_port
```

**Example**

```sh
;peer,ap,192.168.200.1,9999
;peer,station1,raspberrypi.local,9999
```

In the example above, there are two peers:

* An access point with IP address `192.168.200.1` that runs the Wi-Fi Test Suite Linux Control Agent on port 9999
* An device hostname `raspberrypi.local` that runs the Wi-Fi Test Suite Linux Control Agent on port 9999

## Sleep command

The `sleep` command can be used to pause or delay the execution for a specified amount of time.

**Format**

```csv
sleep;duration
```

The `duration` is expressed in seconds

**Example**

```sh
sleep;30
```

## CAPI control command

CAPI control command are prefixed with the peer name.

**Format**

```csv
peer_name;CAPI_command
```

**Example**

```sh
station1;device_get_info
```

Send `device_get_info` command to device `station1`

Please refer to the [CAPI specification](http://www.wi-fi.org/file/wi-fi-test-suite-control-api-specification-v831) resource for CAPI commands list.

### Command parameters

The results from previous commands can be used as parameters for new ones.

**Example**

```sh
station1;traffic_send_ping,destination,192.168.200.1,framesize,1000,frameRate,4,duration,30
;sleep,30
station1;traffic_stop_ping,streamID,<streamID>
```

In this example, the returned `streamID` from command `station1;traffic_send_ping,destination,192.168.200.1,framesize,1000,frameRate,4,duration,30` will be used as an argument for command `traffic_stop_ping`

## Store

`store` command, save a key, value couple to the cache. Those values may be used later as parameters for CAPI commands.

**Format**

```csv
;store,peer_name,key,value
```

**Example**

```sh
station1;sta_is_connected,interface,wlan0
;store,station1,is_connected,<connected>
```

In the example above, the value of `connected` (that is returned by `sta_is_connected` command) will be saved as *is_connected* to the cache.

## Asserts

By default, the program will stop on CAPI failure.

You can add an assert to validate a command result.

**Format**

```csv
;assert,peer_name,operation,key,value
```

**Example**

```sh
station1;sta_is_connected,interface,wlan0
;assert,station1,eq,connected,1
```

In the example above, the program will exit if *connected* is differnt from 1.

### Supported operations

* `eq` assert that the provided value is equals to the current value
* `lt` assert that the provided value is less than the current value
* `lte` assert that the provided value is equals or less than the current value
* `gt` assert that the provided value is greater than the current value
* `gte` assert that the provided value is equals or greater than the current value

## Check

The `check` command is the same as the `assert` one, but instead of causing the program to exit on error, it shows an error message instead.

**Format**

```csv
;check,peer_name,operation,key,value
```

**Example**

```sh
station1;sta_is_connected,interface,wlan0
;check,station1,eq,connected,1
```

In the example above, the program will display an error message *connected* is differnt from 1.

# Start AP example

The sample below creates an access-point with SSID *MySSID* and passphrase *1234567890*

```sh
cat test.txt
```

```sh
# Peers definition
;peer,ap,raspberrypi.local,9999
;new_line

# Configure & start Access Point
ap;ap_set_wireless,NAME,SAMPLE,CHANNEL,6,SSID,MySSID,MODE,11ng
;sleep,2
ap;ap_set_security,NAME,SAMPLE,KEYMGNT,WPA2-PSK,PSK,12344567890
;sleep,2
ap;ap_config_commit,NAME,SAMPLE
;sleep,2
;new_line
```

```sh
./executor -f test.txt
[!] Peers definition
[?] Connecting to <ap> raspberrypi.local:9999
[+] Connected to <ap> raspberrypi.local:9999

[!] Configure & start Access Point
[~] ap <<< ap_set_wireless,NAME,SAMPLE,CHANNEL,6,SSID,MySSID,MODE,11ng
[+] ap >>> status,RUNNING
[+] ap >>> status,COMPLETE
[~] ap <<< ap_set_security,NAME,SAMPLE,KEYMGNT,WPA2-PSK,PSK,1234567890
[+] ap >>> status,RUNNING
[+] ap >>> status,COMPLETE
[~] ap <<< ap_config_commit,NAME,SAMPLE
[+] ap >>> status,RUNNING
[+] ap >>> status,COMPLETE
```
