# Introduction
Hello future me, and maybe people who are here from my resume?

This is a CRBasic program for the autosampler I am building. Here are my goals at this moment, and hopefully I will be good about documenting my progress. Moving my Obsidian document onto here because I would like access on multiple computers.

# Goals of the program
### control pump and valves 
- Pump control: turn pump on and off
- Valve control: open and close NC valves. The sampling valve and pump can activate at the same time, but the valve should close before the pump does. The valve I am currently using is a US Solid motorized ball valve that takes a couple seconds to open or close, so I will have to experiment with the timing there
-  A purge function will be implemented later, but we don't have a vent right now, so they will open at the same time.
- Both the pump and valve (only one is being used right now) are using songle relays for activation. The valve is on C1 and the pump is on C2 of the CR3000
  
### read pressure sensor
- The pressure sensor covers a range of 0 to 145 psig. The recorded pressure is then converted to a voltage between 1 and 4 V, so a function should be created for pressure readout such that:
$y = \frac{145}{4}(x-1)$,
where $y$ is the pressure and $x$ is the output voltage. The output is read through the campell's first analog channel.

### take a sample
- My initial goal was to trigger an interrupt at a given pressure threshold and stop the pump, or shut the system down if that threshold wasn't achievable in time. However, I realized that it would make more sense to calculate when the rate of change of the pressure approaches the noise threshold. I am not sure how this will go when I am pumping into a can that is at vacuum, but I might just implement a silly fix and wait a certain amount of time before the exponential smoothing function takes over
