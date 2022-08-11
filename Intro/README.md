Recapping today quickly (07/07/2022) – it was loosey-goosey so nothing overly serious.

I’ll be taking light notes every talk and recapping them via email/here

# What really is 3G/4G/5G/etc?
-	Just a series of documents covered by an international committee called “3GPP”
      - Consists of random smart people from companies like Verizon, AT&T, Nokia, etc
-	4G/5G are mostly in the ["36 series"](https://www.3gpp.org/DynaReport/36-series.htm?device=desktop)
      - You can see.. a ton of documents
      - Most wireless engineers tend to only focus in one “area” at a time, so it’s more manageable 
         - They are referred to as “L1/L2/L3”. Each “level” gets further from the hardware/radio, and more into the networking/data management.
            - “L1” is more of the “EE side”. Mostly FPGA/RF style of work, but ~40% of it is C++ on normal machines. I find that ~40% super sexy – it’s a lot of high-performance math related code
            - L2/L3 is more on the ‘networking side’
      - Networking can/is also very fun – but it’s closer to straight programming than actual comms theory
      - The fun part is making code super performant; the not-fun part is networking

# What is spread spectrum?
-	Math
-	https://www.ni.com/en-us/innovations/white-papers/06/understanding-spread-spectrum-for-communications.html is actually a really good overview. It’s a bit low-level and not overly informative, but it covers a lot of important keywords to further look into
      - E.g. gold sequences are super important. Direct Sequence Spread Spectrum (DSSS) is what my Matlab project is all about (I’ll cover the why when we deep-dive it next week!). 
      - M-sequences/shift-registers are also super important! They are the foundation of “PseudoNoise” (PN) which is what allows us to recover our signal. It looks like noise, feels like noise, but because it’s entirely faked – we can undo it.
      - Shift-registers are definitely an FPGA thing, but the general idea is still good to know. They are almost essentially the basis of the entire field of cryptography. “Good” cryptography is all about entropy, and shift registers ae controlled entropy. “Real” cryptography takes the concept a bit further, but unless you want a PHD it’s fine to ignore that extent of it.
-	The wiki page (https://en.wikipedia.org/wiki/Spread_spectrum) is actually also really good. Mainly the ‘Telecommunications’ area
      - Frequency hopping, as you had mentioned earlier, is another field of spreading (called ‘FHSS’)


# Docker
-	You’ve worked with the command-line-interface (CLI) so I’ll be brief –
-	“But it works on my machine!”
      - Shut up and send me your machine then
         - This is docker (in basic spirit, at least)
-	Ability to make a lightweight fake-computer that can run anywhere docker is installed
-	Based on Linux, Linux is more precisely “GNU + Linux”
      - GNU is what we interact with
      - “Linux” is more precisely the ‘linux kernel’
-	Docker shares the linux kernel with the machine running it
      - Why is this cool?
         - Comms (& anything requiring deterministic performance) often requires something called a “preemptive” kernel, or in simpler words a “real-time kernel”
            - Comms (& anything requiring deterministic performance) also require some careful tuning after this.. but that’s getting pretty niche and best described by somebody with a long, grey, beard
               - Determinism is critical for consistently meeting deadlines, and thus critical for comms

# Kubernetes
-	What happens if a docker container dies randomly?
      - Well.. I’d need to restart it
         - Enter: Kubernetes (called k8s.. 8 for 8 letters between k and s)
         - K8s will automatically manage this, so it could automatically restart for you
            - (lets us write garbage S/W.. if it crashes no worries – k8s has our back)
         - There is another form called ‘k3s’ for ‘smaller k8s’
            - Same schtick, smaller file size/lighter weight
-	What happens if I need to run 5 containers for one “mission” but 10 for another? (say, a rural vs city cell tower)
      - Well.. I’d need to make a script to launch containers based on some configuration..
         - Enter: (you get the gist)
         - This is the ‘pod’
-	What happens if I want to control the applications on cell-towers in Amish country vs Philadelphia? (many rural towers vs many city towers)
      - Well.. script of scripts
         - .. you get the idea
         - This is a ‘cluster’

Kubernetes is a deceitfully complex tool used to manage (docker) containers

-  What happens if I want to control kubernetes?
      - Enter: "Rancher:" (and "helm charts")
      - Above my pay-grade, but it's very poorly described as a fancy interface to manage kubernetes.. which manages containers..
