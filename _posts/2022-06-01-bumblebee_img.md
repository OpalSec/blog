---
title: "Analysing BumbleBee ISOs, IMGs, and LNK files"
overview: "How to mount and analyse BumbleBee's ISO > LNK + DLL delivery mechanism, plus getting some quick wins through canned tooling and automated sandbox results."
categories:
  - Malware Analysis
tags:
  - lnk
  - IMG
  - ISO
  - capa
  - BumbleBee
  - EXOTICLILY
  - TA578
classes: wide
toc: true
toc_label: "Can't Wait? Jump To..."
toc_icon: "list"
---

> References:
> - Malware Bazaar Sample: [https://bazaar.abuse.ch/sample/666a129f068842ce954380a6de2aa0b5b508f7ed76033c9a8baa48916d193cb3/](https://bazaar.abuse.ch/sample/666a129f068842ce954380a6de2aa0b5b508f7ed76033c9a8baa48916d193cb3/)

This BumbleBee sample was originally delivered [via OneDrive](https://twitter.com/k3dg3/status/1530212471453138947?t=ul2oSDCtpXfMq8dHTSvg1w&s=03) - one of multiple delivery mechanisms adopted by TA578/EXOTICLILY. 

## Background

BumbleBee is a loader malware that first came to prominence when [Google's TAG reported](https://blog.google/threat-analysis-group/exposing-initial-access-broker-ties-conti/) on its use by an Initial Access Broker (IAB) they tracked as EXOTICLILY. This actor was reported to be an IAB working closely with FIN12 (Mandiant, FireEye) / WIZARD SPIDER (CrowdStrike), and appeared in March this year to be favouring this new malware strain over BazarLoader for use in operations. 

[Proofpoint](https://www.proofpoint.com/au/blog/threat-insight/bumblebee-is-still-transforming) later released a post on a BumbleBee campaign they tracked, and attributed to TA578 who they have tracked since May 2020 and performs "email-based campaigns delivering Ursnif, IcedID, KPOT Stealer, Buer Loader, BazaLoader, and Cobalt Strike." BumbleBee appeared to have become the malware of choice for this actor, who distributed it through a variety of means - malspam with malicious URLs/HTML attachments, hijacked email threads with malicious zip archives, and abuse of website contact forms.

Among these campaigns, one thing remained constant - the malware was delivered in a disk image file (ISO/IMG) which contained the BumbleBee dll and an lnk file to execute it. In recent campaigns they have started using IMG files instead, so if you're not familiar with how they work or how to analyse them - this one's for you!

## Parsing the IMG & LNK files

After downloading the sample, we unzip with `7za x \<imgfile\>` and enter the password infected at the prompt to extract the img file:

![file_img](https://opalsec.github.io/assets/images/bumblebee_img_666/file_img.png)

Now that we've got the IMG file, we can mount it using the loop driver using:

```
$> mkdir /tmp/img
$> mount -o loop <imgfile> /tmp/img
```

And next we check to confirm the filetype, and also run exiftool on the lnk file to see what it's trying to do:

![file_lnk](https://opalsec.github.io/assets/images/bumblebee_img_666/file_lnk.png)

Funnily enough it uses 7za to extract the payload like I did for the IMG (great minds think alike?), and drops the extracted docs.dll into %PROGRAMDATA% before executing a specific entry point with rundll32.

The odd thing is that it mentions 7za.exe and a docs.7z - but where are they? According to Remnux, there's just the lnk file in that ISO - nothing else!

![lslah](https://opalsec.github.io/assets/images/bumblebee_img_666/lslah.png)

Turns out not everything can be done in linux - copying this across to my Windows VM and mounting the IMG file, there's still just the lnk sitting there. However, once we tick the magic "Hidden items" checkbox...

![hidden_files](https://opalsec.github.io/assets/images/bumblebee_img_666/hidden_files.png)

As expected - it's trying to drop and run the malicious dll through execution of the lnk file:

![lnk_cmd](https://opalsec.github.io/assets/images/bumblebee_img_666/lnk_cmd.png)

## Extracting the dll file

Next question - I want to check out that dll, how can I extract it?

Sometimes it's as simple as removing the execution component of the command, but unfortunately in this case the permissions on the IMG won't allow it:

![permissions](https://opalsec.github.io/assets/images/bumblebee_img_666/permissions.png)

So on to the next best thing - copy the command line out and run it manually so the file is extracted and dropped to disk (but not run!):

![extract](https://opalsec.github.io/assets/images/bumblebee_img_666/extract.png)

And ta-da! We have the malicious dll

![dll_dir](https://opalsec.github.io/assets/images/bumblebee_img_666/dll_dir.png)

## First-pass analysis

A quick check in [PEStudio](https://www.winitor.com/download) and we can see it contains 15 blacklisted API calls that would allow it to modify, delete and write files and terminate processes, among others

![functions](https://opalsec.github.io/assets/images/bumblebee_img_666/functions.png)

The inbuilt VirusTotal check also helpfully flags that 38 vendors have flagged this as malicious, including Alibaba, who miraculously had a signature for Bumblebee 1099 days ago?

![vt](https://opalsec.github.io/assets/images/bumblebee_img_666/vt.png)

To get more detail on what the dll is capable of, we can run Mandiant's awesome tool [Capa](https://github.com/mandiant/capa/releases) to do some automated analysis:

![capa](https://opalsec.github.io/assets/images/bumblebee_img_666/capa.png)

Capa runs hundreds of inbuilt rules (or you can specify your own) to identify potentially malicious behaviours for a given executable, and that's exactly what it's done here.

We can see a few of the API calls flagged by PEStudio have also been called out here - namely file manipulation and process termination capabilities.

On top of that, it also flags that the malware is capable of deploying some anti-behavioural analysis measures, namely aimed at VirtualBox - but how? 

## Automated Sandbox results & validating assumptions

As good as manual analysis is, automated sandboxes have evolved a lot over the past few years and these days are capable of turning up critical information at-a-glance.

[Triage](https://tria.ge/220527-sxh5taffdp/behavioral6) is one of my favourites, as it runs each sample through a Windows 7 and Windows 10 VM, extracting key information like process tree, C2 details, and more.

![triage](https://opalsec.github.io/assets/images/bumblebee_img_666/triage.png)

Here in the Signatures section, we can see that multiple VirtualBox enumeration signatures were triggered…but unfortunately not the keys which were themselves enumerated.

![signatures](https://opalsec.github.io/assets/images/bumblebee_img_666/signatures.png)

While I submitted a feature request for that to be changed, we can still conduct a manual search in the Registry tab to see if there are any keys read that might related to VirtualBox - and yes there are:

![reg_read](https://opalsec.github.io/assets/images/bumblebee_img_666/reg_read.png)

If you recall the Proofpoint report I mentioned earlier, their in-depth analysis showed that the samples they analysed utilised code stolen from the Open Source [Al Khaser](https://github.com/LordNoteworthy/al-khaser/blob/06399c26a488c1bbdea29fe2023cf5360b640bb7/al-khaser/AntiVM/VirtualBox.cpp) suite, which performs checks for VM artifacts. 

If any of these checks pass, the malware will at best - not run, and at worst - mess with the analyst by popping dummy processes, executing a decoy function, and just generally make you wish you'd never tried analysing it to begin with.

Searching within the repository, we can see this code will indeed check for multiple registry keys indicating the presence of VirtualBox - the same ones found in the Triage results:

![vbox_keys](https://opalsec.github.io/assets/images/bumblebee_img_666/vbox_keys.png)

## Conclusion

Our *manual analysis* of the IMG and LNK files:
1. Gave us the command line used to extract and execute the BumbleBee malware sample;
   - This can be used in conjunction with endpoint telemetry to sweeps a network for Indicators of Activity (IoA) related to EXOTICLILY;
   - When incorporated as part of an ongoing Threat Mapping process, it builds a picture over time of the Delivery portion of the Kill Chain, which SOC analysts can use to identify and triage alerts on anomalous command line and process activity
2. Allowed us to extract the BumbleBee dll for further analysis.

The *first-pass analysis* of the dll was a quick way to get an idea of the capabilities of the malware, and highlighted its capabilities which - if it had successfully detonated - would have provided Incident Responders additional IoAs to pivot on in order to identify other potentially affected devices and ensure eradication from the network.

A review of *automated sandbox results* provided us with C2 IPs to block and prevent any potential further stages from communicating back with the attacker, and highlighted specific registry values interacted with by the malware which we can leverage to create hunts and detections for. 

I should note that as valuable as information from automated sandboxes are - there won't always be results to refer to, and some sandboxes aren't capable of processing certain strands of malware or mitigating certain anti-analysis techniques. This is why it's essential for defenders to be able to perform manual analysis and pivot off public reporting to extract as much information as they can to enable Incident Response, Hunt, and Detection Engineering functions.