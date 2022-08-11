Recap of today (07-18-2022)

# 4G "Acquisition" (all of the fun it takes to actually send data)
To recap our path -- we started with the basics (e.g. cell phone powers on) and then spoke to the "RACH" or ( (Physical) Random Access CHannel) -- more simply put as 'the first time the cell-phone actually does anything'

**Note** all of the below is focused on FDD (Frequency Division Duplexing) 4G and not TDD (Time "" ""). I'm less familiar with TDD other than it is quite different.

SO, with that, the basic steps to acquisition (focusing on 4G, as as we discussed - 5G involves beam-forming and is a fair bit more complex, although a very similar concept)
1. eNB transmits the PSS continuously (Primary Synchronization Signal)
   - UE immediately starts looking in known locations for it'
     - The PSS is always at DC and on the last symbol of slots 0 (subframe 0) and 10 (subframe 5) 
   - The PSS will help with subframe, slot, and symbol synchronization (time domain), the center frequency/carrier frequency (freq domain), and *narrows down* the "Cell Identity"
      - It cannot help with *frame* (10ms / 10 subframes) synchronization as it transmits twice in the same frame, at the same time every time (e.g. consider Frame #1 slot #0 PSS, Frame #0 slot #10 was 5ms in the past, Frame #1 slot #10 is 5ms in the future. Without knowing the first PSS (Frame #1, slot #0) was turned on, how can we tell what frame the next PSS is in? )
2. eNB transmits the SSS continuously (Secondary Synchronization Signal)
   - UE starts looking
      - SSS is also DC, and on the **second to last** symbol of slots 0 (subframe 0) and 10 (subframe 5)
  - The SSS will allow for subframe (1ms) level synchronization, and determine the cell identity
3. eNB transmits the MIB (Master Information Block) continuously (on the BCH or ((Physical) Broadcast CHannel) )
   - UE starts looking know that it knows frames, subframes, slots, and symbols
      - The BCH occupies the first 4 symbols of the 2nd slot (subframe 0) of **every frame**. So it is only transmitted once every 10ms, and retransmitted 4 times
        - So the BCH in entirety is sent every 40ms, but retransmitted 4 times in-between 
   - The MIB carries configuration for:
      - Downlink bandwidth (how many resource blocks)
      - PHICH configuration
         - I had mentioned 'HARQ' (Hybrid Automatic Repeat Request) a few times (calling it one of the most important things in the protocol). PHICH is the channel which HARQ is sent on.
         - Recap: 'HARQ' is an "intelligent retransmission" (more on the MAC notes)
     - Frame number
        - SSS made us aware of the *concept* of frames, but we did not know what the count was. This is the count
      - SIB scheduling information (how to decode the next things!)
 - Recap: At this point -- the PSS allowed us to decode the SSS, which allowed us to decode the MIB, which will allow us to decode more things!
4. eNB will transmit the SIB(s) (System Information Block) continuously on the same channel
   - UE... knows where to find it
   - SIB scheduling is more complicated, and there are 11 of them (though only SIB1 and SIB2 are important for now). SIBs are dynamic (MIB is not) so they are generally transmitted on different intervals
     - SIB1 every 8 frames (80ms). SIB2 every 16 frames (160ms). SIB3 every 32 frames.. etc
   - SIB1 carries information on:
     - Whether or not the UE can access the eNB (e.g. a Verizon phone cannot access AT&T tower unless it pays extra or is dialing 911)
     - Scheduling information on all other SIBs
     - Less interesting fields like the 'Cell ID', Timing Area Control.. higher level fields
   - SIB2 carries information on:
     - Control (where data is, how the channels are configured, etc) & Shared (user data) information/configuration
     - RRC (Radio Resource Control)
     - Uplink power control (used in the RACH)
     - Preamble power ramping (Preamble = RACH waveform. Power ramping = "my RACH failed, let me bump the juice up")
     - Uplink CP (Cyclic Prefix) length
     - Subframe (frequency) hopping patterns
     - "EARFCN" (E-UTRA Abnsolute Radio Channel Number)
       - Alias for Center/Carrier frequency (for Uplink)
- Note: SIB2 has been decoded, so our UE (cell phone) can now do something!
  - For the remaining SIBs, https://www.rfwireless-world.com/Terminology/LTE-MIB-SIB-system-information-blocks.html has an overview of them. Honestly nothing worth covering here, at least IMO. They're important but really not worthwhile.
5. UE transmits RACH ((Physical) Random Access Channel)
- Note: The PSS, SSS, and RACH are all "zadoff chu" signals, which are **not** modulated
  - http://sharetechnote.com/html/RACH_LTE.html#PRACH_Signal_in_Time_Domain is actually exactly what I was looking for the other week
- RACH is the UE transmitting a "preamble", knowing that the eNB will know how to decode it
  - There are 64 possible preambles the UE can choose
  - The eNB will perform a correlation against the received RACH signal, and every preamble
    - (Some vendors will throw in additional preambles to aid in peak detection)
  - When the eNB finds a desirable correlation peak (ideally 1), it will be able to measure the time between the received signal (uplink) and the 'ideal' signal
    - This allows the eNB to determine how far off synchronization the UE is on uplink
6. eNB sends back a RAR (Random Access Response)
    - The eNB will match the difference in peaks to a time duration (e.g. if a UE is 10 km from an eNB, it's expected that the eNB will receive a RACH 33ms after the UE sends it, and the UE will need to send new signals 33ms earlier)
    - The RAR will also provide an "uplink grant", which now allows the UE to start sending "real" data
- Note: on the topic of security -- up until now (bi-directional comms), the data being sent has been **very** insecure.
- The UE does use a temporary identifier (called a GUPTI) to compensate, but there are still risks here within 4G
  - 5G does a better job at securing these initial signals, and thus adds complexity
7. RRC connection request / response
    - The UE will respond to the RAR by requesting a proper (secure) authentication channel
    - The eNB will respond with a full registration to the UE, including a proper ID, and now our communications can be secure!
    - This takes place in the 'Control Plane' / RRC layer. It is primarily in-place to protect against "contention" for registration
       - If multiple UEs attempt to RACH at the same time, they will potentially both see *the same* RAR
       - Both UEs will then respond back with an RRC connection request, but the RAR was only intended for one of them!
       - This is contention. The g/eNB will decide which RRC request is the proper one, and only allow that UE to register

After registration, the UE and eNB are free to pass data back and forth until the UE becomes 'idle', in which case we repeat this process again.

Idle intuitively is just a way to save battery, where the UE does the bare minimum to maintain some level of awareness incase it needs data transfer again.