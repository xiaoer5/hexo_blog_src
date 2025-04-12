---
title: 802.11be phy ppdu structure
date: 2025-04-10 07:32:30
tags:
    - [802.11be]
    - [wifi7]
categories:
    - [ieee802.11, EHT]
    - [ieee802.11, 11be]
    - [ieee802.11, wifi7]
---


EHT STA the structure of PPDU is determined by the TXVECTOR parameters

- **FORMAT**: determines the overall structure of the PPDU
  - Non_HT: 11b/11g/11a (including non-HT duplicate)
  - HT_MF: 11n
  - HT_GF: 11n
  - VHT: 11ac
  - HE_SU: 11ax SU
<!--more-->
  - HE_ER_SU: 11ax ER SU
  - HE_MU: 11ax MU
  - HE_TB:  11ax TB
  - EHT_MU: 11be SU/MU(one or more PSDUs to one or more users)
  - EHT_TB: 11be TB (a single PSDU and is sned in response to a PPDU that carries a triggering frmae)
