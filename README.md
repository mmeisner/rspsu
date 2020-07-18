# Command-line Control of Rohde & Schwarz HMC804x PSUs

This is a simple Python3 command-line utility that can remote control
Rohde & Schwarz HMC8041, HMC8042 and HMC8043 PSUs (from here-on called HMC804x)

The script has no dependencies apart from the Python3 standard library.

Additionally, if you have `zeroconf` installed it can detect and list
instruments on the local network (using zeroconf)

## What can it do?

  - For each output channel:
    - turn on or off
    - set voltage
    - set maximum current
    - toggle (turn off, wait 500ms, then on)
  - Print PSU overall status (default command)
  - Turn on/off master enable
  - Measure current on some channel for given time or number of reports
  - Sleep some time (between commands)

You can issue multiple commands on the command-line, e.g. configuring voltages
for three channels and turning on master enable:

```bash
$ rspsu master=0 v1=3.7 v2=5 v3=4 master=1
```

## Installation

On Linux: Copy the script to somewhere in your PATH (e.g. `$HOME/bin`) and `chmod +x rspsu`.

On windows, copy it to some directory in your PATH


## Examples

```bash
# Get status
$ rspsu
Master: ON
Chan 1: ON   3.700 V  3.000 A  CUR: 1.184 A
Chan 2: ON   5.000 V  3.000 A  CUR: 0.966 A
Chan 3: off  4.000 V  1.041 A  CUR: 0.000 A

# Turn output channel 1 off
$ rspsu off1

# Change output voltage of channel 3
$ rspsu v3=3.2

# Perform 10 current measurements (if current change > 3%)
$ rspsu mi2,20
Measuring current for 9999s, report if change > 3%, 10 reports
Time               dt   I/mA dI/mA d/pct
14:01:26.105   0.000s    638    +0    0%
14:01:27.664   1.560s    742  +105   16%
14:01:27.935   0.271s    644   -98  -13%
14:01:28.145   0.210s    709   +64   10%
14:01:28.960   0.815s    731   +22    3%
14:01:29.683   0.723s    759   +28    4%
14:01:30.458   0.775s    704   -55   -7%
14:01:30.677   0.219s    780   +76   11%
14:01:30.990   0.313s    737   -43   -6%
14:01:31.197   0.207s    867  +129   18%

# Change output voltage, sleep 1s, measure current (3 times)
# and with verbose output so the SCPI commands are printed
$ rspsu v3=3 w=1 mi3 v3=3.1 w=1 mi3 v3=3.2 w=1 mi3 -v
send: *IDN?
recv: Rohde&Schwarz,HMC8043,034889436,HW42000000,SW01.400
send: INST OUT3
send: VOLT 3.000000
send: SYSTem:ERRor?
recv: 0,"No error"
wait: 1.0s
send: MEASure:CURRent?
recv: 0.0000E+00
14:06:48.733   0.000s      0    +0    0%
send: VOLT 3.100000
send: SYSTem:ERRor?
recv: 0,"No error"
wait: 1.0s
send: MEASure:CURRent?
recv: 0.0000E+00
14:06:49.753   0.000s      0    +0    0%
send: VOLT 3.200000
send: SYSTem:ERRor?
recv: 0,"No error"
wait: 1.0s
send: MEASure:CURRent?
recv: 0.0000E+00
14:06:50.792   0.000s      0    +0    0%
213ms

# List all instruments using zeroconf 
$ rspsu -l
Found Rohde&Schwarz HMC8043._lxi._tcp.local.
      Address: 192.168.2.21:80
      Server:  HMC8043.local.
      Props:   textver=1 Manufacturer=Rohde&Schwarz Model=HMC8043 SerialNumber=012345678 FirmwareVersion=01.400
```


## Adding Your Own Functions / Commands

You can add new commands easily by adding a new method to the
`PSUCmdFuncProxy` class. The new method will automatically be available
as a command-line command.

## Adding a PSU Model From Another Vendor 

The HMC804x SCPI command API is simple and thus the `RohdeSchwarzHMC8043`
class that implements the actual SCPI operations over TCP is also simple.

It should be relatively easy to replace the `RohdeSchwarzHMC8043` class
with one for another PSU instrument.
If you do that, please contact me and I will add a link to it
in this README!

## Adding Another Type of Instrument

If you want to modify this script to work with an entirely different
instrument, you need to replace both the `RohdeSchwarzHMC8043` and
the `PSUCmdFuncProxy` classes.
