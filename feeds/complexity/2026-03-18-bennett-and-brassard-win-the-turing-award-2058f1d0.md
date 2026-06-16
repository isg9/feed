---
title: Bennett and Brassard Win the Turing Award
url: https://blog.computationalcomplexity.org/2026/03/bennett-and-brassard-win-turing-award.html
published: "2026-03-18T14:23:34Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-5927640531093807966
---

# Bennett and Brassard Win the Turing Award

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgaLXDID5ulOriI9rziSA_aH4yPGhaHi4iQW6PRy18rH-MgW4Q0ARubN0AjR3BMZ627ion_MGZVNBoYFgFzPYIK-FXIyafkOsFjz0FVnydZdNMOQcOPaH7vJyIvXer4rw9UsZggXwOuJ_ZEfNUFYjlNhuSRKtZS_JoJRVT8TZVCQbfNU5Xc7Enc/s320/HDrrxhAXAAELhHm.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgaLXDID5ulOriI9rziSA_aH4yPGhaHi4iQW6PRy18rH-MgW4Q0ARubN0AjR3BMZ627ion_MGZVNBoYFgFzPYIK-FXIyafkOsFjz0FVnydZdNMOQcOPaH7vJyIvXer4rw9UsZggXwOuJ_ZEfNUFYjlNhuSRKtZS_JoJRVT8TZVCQbfNU5Xc7Enc/s900/HDrrxhAXAAELhHm.jpg)Gilles Brassard and Charlie Bennett

Charlie Bennett and Gilles Brassard will [receive the 2025 ACM Turing Award](https://awards.acm.org/about/2025-turing) for their work on the foundations of quantum information science, the first Turing award for quantum. Read all about it in The [New York Times](https://www.nytimes.com/2026/03/18/technology/turing-award-winners-quantum-cryptography.html?unlocked_article_code=1.UFA.1F_Q.vNM_FfMmBHAQ&smid=url-share), [Science](https://www.science.org/content/article/computer-science-s-nobel-prize-goes-quantum) and [Quanta](https://www.quantamagazine.org/quantum-cryptography-pioneers-win-turing-award-20260318/).

Bennett and Brassard [famously met in the water](https://blog.computationalcomplexity.org/search?q=brassard#:~:text=Gilles%20Brassard%27s%20origin%20story%20of%20quantum%20cryptography.) off a beach during the 1979 FOCS conference in Puerto Rico. That led to years of collaboration, most notably for their [quantum secure key distribution protocol](https://doi.org/10.1016/j.tcs.2014.05.025). The basic idea is pretty simple and does not require quantum entanglement. Alice sends a series of random bits, either straightforward or rotated 45 degrees. Bob measures each of these bits in randomly chosen bases. They discard the bits where they used different bases and use some of the remaining bits to check for eavesdropping, which would collapse the state, and others to set the key.

Bennett and Brassard and four other authors [showed how to teleport](https://doi.org/10.1103/PhysRevLett.70.1895) a quantum bit using entanglement and two classical bits. Bennett with Stephen Wiesner gave the [dual superdense coding protocol](https://doi.org/10.1103/PhysRevLett.69.2881) of sending two classical bits using a single quantum bit.

Bennett and Brassard, with Ethan Bernstein and Umesh Vazirani, [showed](https://doi.org/10.1137/S0097539796300933) that in black-box setting, quantum computers would require \\(\\Omega(\\sqrt{n})\\) queries to search \\(n\\) entries, matching Grover's algorithm. For some reason, the popular press rarely covers these results that limit the power of quantum computing.

I've had the pleasure of knowing both Bennett and Brassard since the 1980s, both just full of enthusiasm and wonderful ideas.

Let me end by saying I don't see this as a Turing award for quantum computing. Once (if?) we get large scale machines, we'll certainly see Turing awards, if not Nobel Prizes, for Peter Shor, Umesh Vazirani and others.
