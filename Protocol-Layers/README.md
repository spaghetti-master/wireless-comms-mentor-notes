Recap of today (07-11-2022)

# 4G/5G protocol layer overview
https://www.tutorialspoint.com/lte/lte_protocol_stack_layers.htm is the diagram that we looked at.

It is focused on 4G. Luckily there is no real difference in 5G other than _where_ each specific layer is hosted (radio unit, tower, data center, etc).

## L1
### Physical Layer (called the PHY/"fai")
- In 4G, this is almost always in an FPGA ("on the radio")
   - In 5G, the PHY can become the "Low-PHY" and "High-PHY". Low-PHY is more FPGA, High-PHY is more CPU (C, C++ code). The High-PHY is primarily aimed at high-performance bit-oriented code, such as (de)compression ("block floating point" is a common algorithm)
      - The low/high PHY is the most commonly supported "version" of 5G currently. Within 5G, they are called "splits" - and are aimed to provide many different hardware/software options depending on user needs. Radisys is a popular vendor decent writeup [here](https://hub.radisys.com/5g-and-iot/exploring-functional-splits-in-5g-ran-tradeoffs-and-use-cases-2)
- The PHY is responsible for "EE" type of tasks:
   - Tasks between an analog waveform and data bits.
   - E.g. (de)modulation, power control, correlations, FEC (Forward Error Correction), etc
   - More formally - it will convert "transport channels" into an OTA (Over-The-Air) signal
      - 'Transport channels' are lower layer entities in the protocol that perform different tasks. E.g. we talked about 'PRACH' (Physical Random Access Channel) which is used to connect to a cell tower. Another channel example would be the 'PUSCH' (Physical Uplink Shared Channel). Both of those are just bits, but they have higher meaning higher within the protocol.
         - The protocol has many things _like_ an IP packet, called SDUs (Service Data Units). Same deal, but basically every layer has their own version of these packets.

## L2 (also called User Plane Protocols)
- In 4G/5G, this is done in software (C++)
   - Some spots are moving to using Golang, but that's fairly rare still
- In 5G, this is often put onto a MEC (Multi-Access Edge Computing)
   - Fancy word for "beefy server close to user"
      - This can literally be carried with a person. Or it can simply be a server located at/near the closest "hub" for a cell tower - still very close, but a bit higher latency)
   - Again, like the PHY, this may not actually be on a MEC - it depends on the "flavor" of 5G
### Medium Access Control (MAC)
- The MAC will (de)multiplex transport channels to/fro 'TBs' (Transport Blocks) (bits) that the PHY understands
   - It is the initial converter between "bits" to "packets". In this case, a "packet" is called a TB, and a bit less formal than something like, say, an IP-packet. For all intents and purposes, the same - but it's just "organized bits" and not seperated into header/data.
- It also controls things like..
   - Data sizing within each specific channel (e.g. sometimes we need a lot of control (information) data compared to user (fun stuff) data)
   - HARQ (Hybrid Automatic Repeat Request)
      - It's a fancy retransmission. If you properly decoded 50% of a transport block ("packet"), HARQ will allow you to request/retransmit the failed 50% to save a bit of bandwidth
      - Important that it's in the MAC (right above the PHY) as it allows for very fast retransmits (lowering overall latency)
   - Controls some other forms of "fancy" reception, such as DRX (Discontinuous Reception)
      - E.g. when your phone is sitting in your pocket/on your desk. You aren't using it, but it's still receiving data and staying awake. The MAC will receive it, see if there is anything important (e.g. an amber alert) and decide if it needs to do anything

### Radio Link Control (RLC)
- TODO - RLC is a bit weird so I'm skipping it for now
   - Specifically, at a low level it is based on managing Transparent Mode (TM), Unacknowledged Mode (UM), and Acknowledged Mode (AM)
      - Important to a MAC/PDCP engineer, but a bit hard to summarize easily 

### Packet Data Convergence Protocol (PDCP)
- First handler of "user data" / most similar to IP packetization
- Importantly, PDCP will handle (de)compresion and (de)ciphering of the packet (SDU) headers and data, and is the primary "router" for specifically user data
- Importantly, PDCP will reorganize packets (SDUs) into the correct sequence as-transmitted
- Importantly, PDCP will primarily handle "handovers" - where a phone will transfer connection between towers
- Less importantly, the PDCP will decode messages specific to L3 (RRC/NAS) such as SRB /DRB (Signaling Radio Bearer / Data Radio Bearer)
   - SRB carries "signals", or control information
   - DRB carries "user data"
      - There are 2 types of DRB.. maybe a bit in the weeds. Default (1) and Dedicated (2). Default is just normal service. Dedicated is for more niche service like Facetime.
- Side note: (de)compression is actually pretty fun

## L3 (also called Control Plane Protocols)
- This is also software in 4G/5G
   - Still mostly C++ in practice, but Golang is definitely gaining favor here from startups
- In 5G, this is the 5GC (5G Core)
   - It effectively is hosted in "data centers"
      - Like everything else _it depends_, but the 5GC is the brains of the whole show, so it is often accessible by many towers

### Radio Resouce Control (RRC)
- Primarily handles the 'state' behind the network (very simple state machine.. Idle or Connected)
   - When idle:
      - RRC will assess some RF parameters to determine what it could communicate to, and if it needs to be woken up/start sending data (e.g. your phone is idle, but somebody texts you)
   - When connected:
      - RRC will determine high-level performance metrics of the Radio/Air interface, such as the CQI (Channel Quality Indicator -- bad channel = bad quality)
- Handles all broadcast signals
   - Broadcast signals, such as the SIBs (System Information Blocks) contain all information required to connect to a network
- Handles all high-level fields relating to connecting to a network / managing the radio itself
   - Each network establishment "creates" an "RRC". Each network disconnect "releases" an RRC
   - Fancy way of saving battery/power

### Non-Access Stratum security (NAS)
- Handles _all_ encryption of the keys used for secure communications
   - The keys are used to secure things like your SIM card's unique ID (like a static IP), and even used to generate hashing codes (called Gold Codes)
      - Gold Codes are a combination of two bit sequences wich a specific maximum cross-correlation value. They are managed via LFSRs (Linear-Feedback Shift Registers) which are a neat (and fairly simple) concept in cryptography
   - We discussed "PRACH" a bit and I had mentioned ['Zadoff-Chu'](https://en.wikipedia.org/wiki/Zadoff%E2%80%93Chu_sequence)
      - Zadoff-Chu (often just called ZC) is a special case of this where the combined sequences are CAZAC (Constant Amplitude Zero Autocorrelation Waveform)
         - Very important in some cases, as you now have a cryptographically secure signal that also containes positional (e.g. delay over the air) information given the known autocorrelation
- There isn't anything else too-too easy or interesting to cover. NAS is pretty niche, and more often just kind-of there in the form of an AES library


# Code stuff
## C++ stuff
- https://godbolt.org/
   - Online compiler "explorer"
      - Great sandboxing tool, and it shows you the output in ASM (with highlighting to map it back to C++!!)
- Current C++ proposals: https://open-std.org/jtc1/sc22/wg21/docs/papers/2022/
   - There are a lot!
   - https://open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2300r5.html is by far the sexiest as of now
   - https://open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2266r3.html is a bit more simple (HA) and interesting
      - The topic is pretty difficult. "Implicit move" is an optimization performed by compilers as a 'second step' to something called RVO (return value optimization)
         - RVO is super neat. https://godbolt.org/z/o63ocP9ev is a bit of a bad example, but kind of shows how cool it is
- C++ committee structure: https://isocpp.org/files/img/wg21-structure-2021-06.png
- RAII (Resource Allocation Is Initialization)
   - Pretty much just means 'The } is mightier than the sword'
      - Initialize an object, initialize a resource (e.g. a file, memory, socket, etc)
      - Destroy an object, automatically release a resource
         - No need to remember to clean-up!
         - No more resource leaks!
            - Note: the C++ STL (Standard Template Library) is a bit "selfish" with the default memory allocator. There are quite a few common situations where leaks" appear to exist, but are really as-defined
               - If this is interesting - I love me a good allocator and love me this topic
   - https://www.learncpp.com/cpp-tutorial/destructors/ is a pretty great overview
- Compiler support: https://en.cppreference.com/w/cpp/compiler_support
   - Releases every 3 years (on/after 2011)
   - Compilers tend to support the releases 3 years later (so C++11 was fully supported in 2014, C++14 in 2017, C++17 in 2020, etc)
      - This is just a guideline, and depends. The link above is really the truth
