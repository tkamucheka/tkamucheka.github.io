---
layout: project
title: Kyber512 in Hardware
project: kyber512
---

A pure-hardware implementation of the Kyber512 key encapsulation mechanism
(KEM) with CCA security, for FPGAs, and a variant masked against power-based
side-channel attacks. Kyber was selected by NIST for standardization as a
post-quantum KEM.

## Repositories

| Repository | Contents |
|------------|----------|
| [Kyber512_INDCCA2](https://github.com/tkamucheka/Kyber512_INDCCA2) | Vivado project of the Kyber512 CCA-KEM in a system on chip |
| [Kyber512_INDCCA2_Masked](https://github.com/tkamucheka/Kyber512_INDCCA2_Masked) | Vivado project of the masked implementation |
| [Kyber512_INDCCA2_AXI_IP](https://github.com/tkamucheka/Kyber512_INDCCA2_AXI_IP) | The implementation packaged as Vivado IP with AXI interfaces, with its bare-metal driver and test designs |
| [Kyber512_INDCCA2_Masked_AXI_IP](https://github.com/tkamucheka/Kyber512_INDCCA2_Masked_AXI_IP) | The masked implementation packaged as Vivado IP with AXI interfaces |

## Publications

* **Kamucheka, T.**, Nelson, A., Andrews, D., & Huang, M. (2022).
  [A masked pure-hardware implementation of Kyber cryptographic algorithm.](https://ieeexplore.ieee.org/document/9974404)
  In 2022 International Conference on Field-Programmable Technology (ICFPT).
* **Kamucheka, T.**, Fahr, M., Teague, T., Nelson, A., Andrews, D., &
  Huang, M. (2021).
  [Power-based side-channel attack analysis on PQC algorithms.](https://www.semanticscholar.org/paper/Power-based-Side-Channel-Attack-Analysis-on-PQC-Kamucheka-Fahr/0cfd3aa9dfdc232cfa0f9802db72d9d33d4e3c93)
  Cryptology ePrint Archive.

More publications are listed under [Publications]({{ '/Research/' | relative_url }}).
