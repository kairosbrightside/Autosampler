# Introduction
This is a CRBasic program for the autosampler I am building. Here are my goals at this moment, and hopefully I will be good about documenting my progress. Slowly transitioning from Obsidian to GitHub because I prefer the version control + increased access on here, and I wasn't big on Obsidian to begin with

# Hardware
### Components
- CR3000 micrologger (Campbell scientific)
- 8x electronically actuated valves (US Solid)
- Pump (??? ancient and has no identifying markings but it runs on 12V so that's good enough!)
- Drying system 
    - magnesium perchlorate tube (homemade)
    - TECs: 2x 12715, 2x 12715 (generic)
    - Thermal paste (generic)
    - Water cooling system (generic)
    - Thermocouple (Omega Engineering)
- 6x sample cans (0.8 L SUMMA)
  - ideally also one can for zero air/purge gas prior to sampling
- Digital pressure gauge (MCMaster)
- Songle relays (generic)
- Power supply
- 12 $\rightarrow$ 5V voltage divider (homemade)
- CD74HC4067 multiplexing board so we don't need more relays

# Goals of the program
### control pump and valves 
- Pump control: turn pump on and off
- Valve control: open and close NC valves. The sampling valve and pump can activate at the same time, but the valve should close before the pump does. The valve I am currently using is a US Solid motorized ball valve that takes a couple seconds to open or close, so I will have to experiment with the timing there
-  A purge function will be implemented later, but we don't have a vent right now
- Both the pump and valve (only one is being used right now) are using songle relays for activation. The valve is on C1 and the pump is on C2 of the CR3000
  
### read pressure sensor and thermocouple
- The CR3000 is using a 1s scan rate
- The pressure sensor covers a range of 0 to 145 psig. The recorded pressure is then converted to a voltage between 1 and 4 V, so a function should be created for pressure readout such that:
  
$$ y = \frac{145}{4}(x-1) $$

where $y$ is the pressure and $x$ is the output voltage. The output is read through the campell's first analog channel.

- The DC+ should be connected to brown and DC- should be connected to blue!!

- I would also like to read the differential output of the thermocouple, and log the dew point and lowest temperature it reached, but this can wait for now.


### take a sample
- My initial goal was to trigger an interrupt at a given pressure threshold and stop the pump, or shut the system down if that threshold wasn't achievable in time. However, I realized that it would make more sense to calculate when the rate of change of the pressure approaches the noise threshold. I am not sure how this will go when I am pumping into a can that is at vacuum, but I might just implement a silly fix and wait a certain amount of time before the exponential smoothing function takes over

### Sample procedure
1) Open the purge valve (v7) and run air through for 5 minutes (300 seconds)
2) Close the purge valve and open the valve on a given sample can
3) When $\Delta P \to 0$, open purge valve again. Then vent can and fill again (repeat this 2x?)
4) Finally, log final sample pressure and time

### write logged data to CF card
- I hope it is exportable to csv but i am not sure...
- 
