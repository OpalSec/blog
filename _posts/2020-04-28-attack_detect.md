---
title: "ATT&CK & DeTT&CT - Correlating Attack & Defence"
overview: "Make ATT&CK actionable - The ATT&CK Navigator acts as a common frame-of-reference for both Defensive and Offensive datasets. DeTT&CT allows you to map your Defensive apertures against the ATT&CK Matrix, which can in turn be directly triaged against the Offensive TTPs Threat Actors relevant to your Organisation employ. "
categories:
  - Threat Intel
tags:
  - Cyber Threat Intelligence
  - CTI
  - ATT&CK
  - DeTT&CT
classes: wide
toc: true
toc_label: "Can't Wait? Jump To..."
toc_icon: "list"
---

![Banner](https://opalsec.github.io/assets/images/attack_dettect/banner.png)

## What are they?

**The ATT&CK Navigator** is designed to provide basic navigation and annotation of ATT&CK matrices, enabling you to visualize your defensive coverage, your red/blue team planning, the frequency of detected techniques, and more. The principal feature of the Navigator is the ability for users to define layers - custom views of the ATT&CK knowledge base - which can be customised by manipulating the cells in the matrix with color coding, adding annotations, etc.

**DeTT&CT** builds on this visualisation capability provided by ATT&CK Navigator, and can assist blue teams in using ATT&CK to score and compare data log source quality, visibility coverage, detection coverage and threat actor behaviours. Moreover, it can enable analysts to:

- Administrate and score the quality of your data sources.
- Get insight on the visibility you have on endpoints.
- Map your detection coverage.
- Map threat actor behaviours.
- Compare visibility, detections and threat actor behaviours to uncover possible improvements in detection and visibility. This can help you to prioritise your blue teaming efforts.

## Why, and how, can I use them to enhance CTI processes?

The business case for adopting and integrating the ATT&CK Navigator and DETT&CT Framework into CTI processes is quite clear cut:
- ATT&CK Navigator provides a visualisation tool which enables analysts to heatmap Attacker TTPs identified through Incident Response and CTI Analysis and Reporting;
- DeTT&CT can leverage Navigator's visualisation capabilities to map your existing detection apertures against identified Threat Actors and their methodologies, enabling Organisations to qualitatively assess their Defensive posture in relation to their specific threat landscape.

Essentially, the ATT&CK Navigator acts as a common frame-of-reference for both Defensive and Offensive datasets. DeTT&CT allows you to map your Defensive apertures against the ATT&CK Matrix, which can in turn be directly triaged against the Offensive TTPs Threat Actors relevant to your Organisation employ. 

> **NOTE:** This post is just going to walk you through how to set these tools up on an Ubuntu server - I'll hopefully have practical walkthroughs of how to apply this in your organisation up soon!

## Installing the ATT&CK Navigator

### Reference:
https://github.com/mitre-attack/attack-navigator#install-and-run

### Pre-Requisites:

#### *Node.js version 10+*
Documentation says version 8+ will work, but that appears to be outdated:

![nodeversion](https://opalsec.github.io/assets/images/attack_dettect/node_version.png)

To upgrade to the latest stable version, we first need to clear out the npm cache, before installing the [Node Version Manager helper - 'n'](https://github.com/tj/n)

![n](https://opalsec.github.io/assets/images/attack_dettect/n.png)

Then we simply run ```n stable``` to upgrade to the latest stable version:

![nupgrade](https://opalsec.github.io/assets/images/attack_dettect/nupgrade.png)

#### *Angular CLI*

Hopefully you didn't make the mistake I did by installing ng-common as suggested by Ubuntu when you attempt to run the ```ng serve``` command to start up the node.js server. 

However, if you did - just run ```apt purge ng-common``` to uninstall it, before installing the Angular CLI (what the ng serve command is meant to be invoked by in the setup phase) with ```npm install -g @angular/cli```:

![angular](https://opalsec.github.io/assets/images/attack_dettect/angular.png)

### Installation

1. Clone the git repository to your local machine with ```git clone https://github.com/mitre-attack/attack-navigator.git```
2. Once complete, cd into the nav-app directory and run ```npm install```
3. Once that completes, use ```ng serve``` to spin up the local webserver -make sure you've got the pre-requisites installed (Angular CLI) before running it!

Depending on your OS, you might get flooded with error messages like these (I did on Ubuntu):

![errors](https://opalsec.github.io/assets/images/attack_dettect/errors.png)

These can be fixed by manually running ```npm run postinstall``` before re-running ```ng serve``` to start ATT&CK Navigator.

## Installing DeTT&CT

### Reference:
https://github.com/rabobank-cdc/DeTTECT/wiki/Installation-and-requirements#local-installation

### Pre-Requisites:
- Python 3.6+
- Pip3
- attackcti, simplejson, ruamel.yaml, plotly, pandas and xlsxwriter python packages (these can be installed with pip, as shown below)

### Installation

1. This is much more straightforward - just clone the repository with ```git clone https://github.com/rabobank-cdc/DeTTECT```
2. Then cd into the installation directory and install the necessary python packages (if you haven't already got them installed) with ```pip install -r requirements.txt```

## Verifying Everything Works

DETT&CT comes with a bunch of sample data, one of which is a sample YAML file containing information on data source coverage. 

We can generate an ATT&CK Navigator layer with ```python dettect.py ds -fd sample-data/data-sources-endpoints.yaml -l```

![generate](https://opalsec.github.io/assets/images/attack_dettect/generate.png)

And finally visualise it in the ATT&CK Navigator by browsing to the local webserver on ```http://localhost:4200``` and opening the generated layer by selecting "Open Existing Layer" > "Upload from local" and selecting the json file that was generated in the output folder:

![select](https://opalsec.github.io/assets/images/attack_dettect/select.png)

If everything installed correctly, you should see this gorgeous mapping of data sources in your browser:

![map](https://opalsec.github.io/assets/images/attack_dettect/map.png)