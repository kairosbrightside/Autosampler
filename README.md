# Introduction
This is a CRBasic program for the autosampler I am building. Here are my goals at this moment, and hopefully I will be good about documenting my progress. 

# Hardware
### Electrical
- CR3000 micrologger (Campbell scientific)
- 8x electronically actuated valves (US Solid)
- Pump (??? has no identifying markings but it runs on 12V so that's good enough!)
- Digital pressure gauge (MCMaster)
- Songle relays (generic), using an 8-relay board for 7 valves + pump
- Terminal voltage board 8 x 2 for distributing 12V
- Power supply for CR3000 (18V)
### Gas Flow
- Drying system 
    - magnesium perchlorate tube (homemade)
- 6x sample cans (0.8 L SUMMA)
- 3x Swage cross
- 1x Swage tee
  
Optional (in case of need for more)
- CD74HC4067 multiplexing board since the CR3000 only has 8 digital pins


# Goals of the program
### control pump and valves 
- Pump control: turn pump on and off
- Valve control: open and close NC valves. The sampling valve and pump can activate at the same time, but the valve should close before the pump does. The valve I am currently using is a US Solid motorized ball valve that takes a couple seconds to open or close, so I will have to experiment with the timing there
-  Purge the line for 5 minutes before opening the can
- The pump and valves are using songle relays for activation. The sample valves are on C1-6, the purge valve is on C7, and the pump is on C8 of the CR3000
  
### read pressure sensor
- The CR3000 is using a 1s scan rate
- The pressure sensor covers a range of 0 to 145 psig. The recorded pressure is then converted to a voltage between 1 and 4 V, so a function should be created for pressure readout such that:
  
$$ y = \frac{145}{4}(x-1) $$

where $y$ is the pressure and $x$ is the output voltage. The output is read through the campell's first analog channel.

- The DC+ should be connected to brown and DC- should be connected to blue!!

### Stop samples at the limit of the pump
- The program uses an exponential smoothing function to calculate when the pump has hit its limit. This is accomplished by calculating the exponentially smoothed derivative of the pressure, and then stopping when that derivative only deviates by values within the noise threshold of the sensor for several increments in a row.
  
### Sample procedure
1) Open the purge valve (v7) 
2) Turn pump on and run air through sample line for 5 minutes (300 seconds)
3) Close the purge valve and open the valve on a given sample can
4) When $\Delta P \to 0$, open purge valve again. Then vent can and fill again.
5) Once can has been vented 2x, close purge valve and keep can valve open. Fill can.
6) Finally, log final sample pressure and time

### write logged data to CF card
- I think someone still needs to connect their computer to it to actually export the data though...

# Tests before fielding

### 9/15/25 Week-long leak test

On Monday, 9/8, collected 3 samples of roof air 1h apart and held them for a week. On 9/15, removed the samples to measure them. 
Cans 1 and 3, which were at 40.36 and 40.39 psig respectively, maintained their pressures when measured by an analog gauge, but can 2 went down to 31 psi for some reason.

Turning the autosampler on its side to leak-check valve connections did not show any major leaks (at least when checked with Swagelock snoop fluid?). 
However, the source of the large leak after turning off the pump was identified to be the pump itself, as capping the inlet resulted in a significant increase in the time it took for the system to go down from 40 psig, and I felt it with my hand too.
After capping the pump, the leak rate went down to 0.1 psig every 5ish seconds, with the 4 suspect valves (the one whose can failed the leak test and the 3 new ones) open and capped.
Closing these valves resulted in an even faster leak rate of 0.1 psig/3 seconds, which indicates that the majority of the leak is probably within the manifold, and not past one of its valves.

##### 9/8 pressure logs:
"TOA5","CR3000","CR3000","3626","CR3000.Std.09","CPU:prototype.CR3","11936","SampleLog"
"TIMESTAMP","RECORD","V_Pressure","Pressure_psi","State","SampleID","CurrentCan","HourNow","MinuteNow"
"2025-09-08 10\:42:57",0,2.113,40.36,"SHUTDOWN",1,1,10,42
"2025-09-08 11\:42:57",1,2.115,40.41,"SHUTDOWN",2,2,11,42
"2025-09-08 12\:42:57",2,2.114,40.39,"SHUTDOWN",3,3,12,42
