# L.U.N.A. User Manual
L.U.N.A. (Low-cost Unit for Near-background Atmospheric Samples) is an autonomous air-sampling system designed to collect near-background atmospheric samples into SUMMA canisters with minimal user intervention. 
The system is intended for deployment at semi-remote locations where power is available but frequent manual sampling is impractical.
L.U.N.A. uses a CR3000 micrologger to coordinate valve actuation, pump control, and pressure monitoring. 
Sample termination is determined using the rate of change of pressure, rather than a fixed pressure setpoint, allowing robust operation despite sampling condition variability.

### System Capabilities
- Autonomous daily sampling at scheduled times
- Sequential filling of up to 6 SUMMA canisters
- Line purge prior to sampling
- Multiple can purge/vent cycles before final fill
- Derivative-based fill termination (pump-limited)
- Event-based logging to CompactFlash 

### Hardware 
#### Electrical Components
| Component                                    | Purpose                  | Notes                                      | Failure Risk                         |
| -------------------------------------------- | ------------------------ | ------------------------------------------ | ------------------------------------ |
| **CR3000 Micrologger** (Campbell Scientific) | System controller        | Runs CRBasic program, handles timing & I/O | Low                                  |
| **Songle Relay Board (8-channel)**           | Switches pump and valves | cheap, replaceable                         | **High** (most likely failure point) |
| **US Solid Motorized Ball Valves**           | Gas flow control         | Normally-closed                            | Low                                  |
| **12 V DC Pump**                             | Pressurizes canisters    | can be switched out if it fails            | Medium                               |
| **Pressure Transducer (McMaster-Carr)**      | Measures can pressure    | 1–5 V analog output                        | Low                                  |
| **Terminal Distribution Block**              | 12 V power distribution  | Passive                                    | Low                                  |
| **18 V DC Power Supply**                     | Powers CR3000            | From 120 V AC                              | Low                                  |

> Honestly if something fails it's probably the relay board, as songle relays are notorious for failing. It's a 5V 8-channel songle relay board

3.2 Gas Flow Components
- Magnesium perchlorate ${\rm Mg(ClO}_4{\rm )}_2$ tube (homemade)
- 8x electronically actuated valves (US Solid)
    - 6x valves for samples
    - 1x vent valve
    - 1x inlet valve (to keep ${\rm Mg(ClO}_4{\rm )}_2$ dry)
- 6x sample cans (0.8 L SUMMA)
- 3x Swage cross
- 1x Swage tee
- 1x bulkhead fitting for inlet
- 1/4 in stainless steel tubing to construct manifold
- 1/4 in Synflex tubing
> The magnesium perchlorate and the sample can connections should be checked carefully. There may come a day where the manifold connection fittings wear out, which means you wil have to find more 1/4" stainless steel tubing and bend it at a 90-degree angle and attach fittings to it and put it on the manifold

### System Design
#### Digital Outputs (CR3000)
| Port  | Function           |
| ----- | ------------------ |
| C1–C6 | Sample can valves  |
| C7    | Purge / vent valve |
| C8    | Pump               |

All relays are active-high (`PortSet = 1` $\to$ `ON`).

#### Analog Inputs
| Channel | Signal                             |
| ------- | ---------------------------------- |
| SE1     | Pressure transducer output (1–5 V) |

Pressure conversion:
$$P_{\rm psi} = \frac{145}{4}(V-1)$$

### Sampling Logic 

L.U.N.A. does not fill to a fixed pressure. Instead, it stops when the smoothed pressure derivative approaches zero, indicating the pump has reached its limit.
This compensates for pump wear, works even if leaks reduce maximum pressure, and avoids false termination due to sensor noise.

### Sampling Sequence
For each canister:

0) Waiting for sample trigger (`State = "WAIT"`)
  - Initializes sampling sequence when a specific time or trigger occurs
1) Line purge (`State = "FIRST_PURGE"`)
  - Purge valve open, pump on
  - Duration: 5 minutes
2) Can purge cycles (`State = "CAN_PURGE"`
  - Can valve open, purge closed
  - Fill until $\Delta P \to 0$
  - Vent can (`State = "VENT"`)
  - Repeat 2x
3) Final fill (`State = "SAMPLE"`)
  - Can valve open
  - Pump runs until derivative-based stop
4) Shutdown (`State = "SHUTDOWN"`)
- Close valves
- Turn pump off
- Log final pressure and timestamp
- Increment sample nummber
- Return system to `WAIT` state

Only one log row per sample is written, as to not be overwhelming.

### Datalogging
- Data stored on CompactFlash card
- Event-based logging (no continuous clutter)
- Exported as CSV via PC400

#### Logged fields:
- Timestamp
- Pressure (V and psi)
- State
- Sample ID
- Can number
- Hour / minute of sample

### Operating the system
#### Startup Checklist
- Verify valves closed
- Verify pump OFF
- Check CF card is inserted
- Confirm CR3000 clock is correct, set to computer clock with PC400 if not
- Confirm COM connection before deployment
- Ensure the sample cans and magnesium perchlorate connections are not leaking
  - It can help to pressurize the system by turning the pump on, setting all the relays to high, and using leak checking fluid to see if there are any bubbles around those connections, or other ones you've changed (keep the cans closed if you do this though!!)

### Deployment
- Power system from wall rails
- Please don't put LUNA in precarious places. I think it could fall down a hill but I don't want to learn....
- Samples collected automatically at scheduled times
  - You can edit the program to change scheduled times if you want. Instructions for this detailed in program comments.

### Troubleshooting Guide
#### Cannot Connect to CR3000 (COM Errors)
Symptoms:
- “serial open failed”
- “unreachable destination”
- PC400 won’t connect

Fixes:
- Check which COM port appears in Device Manager
- Update PC400 station to match that COM port
- Close PC400
- Turn literally everything off and on again
- Get a new RS232-USB cable... It's really picky...
> Disconnecting batteries can cause COM port reassignment.

#### Valves Not Opening / Pump Not Running
> Likely cause: relay failure

Actions:
- Manually toggle relay channels
- Replace Songle relay module if inconsistent

#### Sample Pressure Drops After Collection
> Please note that a small drop ($<5$ psi) in sample pressure after collection is normal, especially if LUNA was put somewhere sunny. This is due to the gas being sampled being at a higher temperature than it will be when it gets to the lab indoors (the sampling area might be hot, and the pump will be hot when the air passes through it)
Possible causes:
- Pump backflow (no check valve)
- Manifold leak
- Valve internal leakage

Diagnostic steps:
- Cap pump inlet and monitor decay
- Close all valves individually
- Leak-check with Swagelok Snoop by opening every valve in the manifold while running the pump0

#### CF Card Not Recording Data
- Ensure CardFlush() is executed after logging
- Confirm card formatted FAT/FAT32
- The CR3000 is picky about what cards it uses, so try a couple different ones
- Check available space
- Try re-inserting card after power cycle

### Maintenance Recommendations
| Component     | Action            | Interval                |
| ------------- | ----------------- | ----------------------- |
| Songle relays | Inspect / replace | As needed               |
| Sample cans   | Leak check        | Before deployment       |
| Drying tube   | Replace Mg(ClO₄)₂ | Before deployment       |
| Valves        | Cycle test        | If acting suspicous     |

### Known Limitations
- Manual data retrieval required
- Songle relays limit lifetime (feel free to replace with another kind of relay, just make sure they're 5V high-active---convenient for the digital pins of the CR3000---or change the program accordingly!!)
- Needs wall power 

### Revision History?0
- v1.0 — Initial field-ready deployment (balcony test)
