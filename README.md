<!--
 Licensed to the Apache Software Foundation (ASF) under one
 or more contributor license agreements.  See the NOTICE file
 distributed with this work for additional information
 regarding copyright ownership.  The ASF licenses this file
 to you under the Apache License, Version 2.0 (the
 "License"); you may not use this file except in compliance
 with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing,
 software distributed under the License is distributed on an
 "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 KIND, either express or implied.  See the License for the
 specific language governing permissions and limitations
 under the License.
-->
# Apache Airflow

[![PyPI version](https://badge.fury.io/py/apache-airflow.svg)](https://badge.fury.io/py/apache-airflow)
![Airflow](https://github.com/apache/airflow/workflows/Airflow/badge.svg)
[![Build Status](https://travis-ci.org/apache/airflow.svg?branch=master)](https://travis-ci.org/apache/airflow)
[![Coverage Status](https://img.shields.io/codecov/c/github/apache/airflow/master.svg)](https://codecov.io/github/apache/airflow?branch=master)
[![Documentation Status](https://readthedocs.org/projects/airflow/badge/?version=latest)](https://airflow.readthedocs.io/en/latest/?badge=latest)
[![License](http://img.shields.io/:license-Apache%202-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0.txt)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/apache-airflow.svg)](https://pypi.org/project/apache-airflow/)
[![Twitter Follow](https://img.shields.io/twitter/follow/ApacheAirflow.svg?style=social&label=Follow)](https://twitter.com/ApacheAirflow)
[![Slack Status](https://img.shields.io/badge/slack-join_chat-white.svg?logo=slack&style=social)](https://s.apache.org/airflow-slack)

_NOTE: The transition from 1.8.0 (or before) to 1.8.1 (or after) requires uninstalling Apache Airflow before installing the new version. The package name was changed from `airflow` to `apache-airflow` as of version 1.8.1._

Apache Airflow (or simply Airflow) is a platform to programmatically author, schedule, and monitor workflows.

When workflows are defined as code, they become more maintainable,
versionable, testable, and collaborative.

Use Airflow to author workflows as directed acyclic graphs (DAGs) of tasks. The Airflow scheduler executes your tasks on an array of workers while following the specified dependencies. Rich command line utilities make performing complex surgeries on DAGs a snap. The rich user interface makes it easy to visualize pipelines running in production, monitor progress, and troubleshoot issues when needed.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of contents**

- [Requirements](#requirements)
- [Getting started](#getting-started)
- [Installing from PyPI](#installing-from-pypi)
- [Building customized production images](#building-customized-production-images)
- [Beyond the Horizon](#beyond-the-horizon)
- [Principles](#principles)
- [User Interface](#user-interface)
- [Using hooks and Operators from "master" in Airflow 1.10](#using-hooks-and-operators-from-master-in-airflow-110)
- [Contributing](#contributing)
- [Who uses Airflow?](#who-uses-airflow)
- [Who Maintains Apache Airflow?](#who-maintains-apache-airflow)
- [Can I use the Apache Airflow logo in my presentation?](#can-i-use-the-apache-airflow-logo-in-my-presentation)
- [Airflow merchandise](#airflow-merchandise)
- [Links](#links)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Requirements

Apache Airflow is tested with:

### Master version (2.0.0dev)

* Python versions: 3.6, 3.7, 3.8
* Postgres DB: 9.6, 10
* MySQL DB: 5.7
* Sqlite - latest stable (it is used mainly for development purpose)
* Kubernetes - 1.16.2, 1.17.0

### Stable version (1.10.11)

* Python versions: 2.7, 3.5, 3.6, 3.7, 3.8
* Postgres DB: 9.6, 10
* MySQL DB: 5.6, 5.7
* Sqlite - latest stable (it is used mainly for development purpose)
* Kubernetes - 1.16.2, 1.17.0

### Additional notes on Python version requirements

* Stable version [requires](https://github.com/apache/airflow/issues/8162) at least Python 3.5.3 when using Python 3

## Getting started
Visit the official Airflow website documentation (latest **stable** release) for help with [installing Airflow](https://airflow.apache.org/installation.html), [getting started](https://airflow.apache.org/start.html), or walking through a more complete [tutorial](https://airflow.apache.org/tutorial.html).

> Note: If you're looking for documentation for master branch (latest development branch): you can find it on [ReadTheDocs](https://airflow.readthedocs.io/en/latest/).

For more information on Airflow's Roadmap or Airflow Improvement Proposals (AIPs), visit the [Airflow Wiki](https://cwiki.apache.org/confluence/display/AIRFLOW/Airflow+Home).

Official Docker (container) images for Apache Airflow are described in [IMAGES.rst](IMAGES.rst).

## Installing from PyPI

Airflow is published as `apache-airflow` package in PyPI. Installing it however might be sometimes tricky
because Airflow is a bit of both a library and application. Libraries usually keep their dependencies open and
applications usually pin them, but we should do neither and both at the same time. We decided to keep
our dependencies as open as possible (in `setup.py`) so users can install different versions of libraries
if needed. This means that from time to time plain `pip install apache-airflow` will not work or will
produce unusable Airflow installation.

In order to have repeatable installation, however, introduced in **Airflow 1.10.10** and updated in
**Airflow 1.10.12** we also keep a set of "known-to-be-working" constraint files in the
orphan `constraints-master` and `constraints-1-10` branches. We keep those "known-to-be-working"
constraints files separately per major/minor python version.
You can use them as constraint files when installing Airflow from PyPI. Note that you have to specify
correct Airflow tag/version/branch and python versions in the URL.

1. Installing just airflow:

```bash
pip install apache-airflow==1.10.12 \
 --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt"
```

2. Installing with extras (for example postgres,gcp)
```bash
pip install apache-airflow[postgres,gcp]==1.10.11 \
 --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt"
```

## Building customized production images
In order to use Airflow in Docker Compose or Kubernetes, you might need to use or build production images of Apache Airflow. The production image is a multi-segment image. The first segment "airflow-build-image" contains all the build essentials and related dependencies that allow to install airflow locally. By default the image is build from a released version of Airflow from Github, but by providing some extra arguments you can also build it from local sources.
 This is particularly useful in CI environment where we are using the image to run Kubernetes tests(Helm Chart integration). We will also use released images in the Helm Chart(backward compatibility). You can see DockerHub images at https://hub.docker.com/repository/docker/apache/airflow. In DockerHub there is just a convenience binary and that image can (and often should) be built from the officially released sources. More Details [TODO](#).

The community provides two types of support for the production images:
- We provide pre-build released version of production image in PyPI build from released sources of Apache Airflow - shortly after release. Those images are available in the DockerHub. You can pull those images via `docker pull apache/airflow:<VERSION>-pythonX.Y` - version is the version number (for example 1.10.11). Additionally `docker pull apache/airflow` will pull latest stable version of the image with default python version (currently 3.6). Now change to root user(here we use root user) temporarily and install your own dependencies, for example mx. Change back to airflow user and then you can install some pip packages you want. You can add your dags by copying them and then build your own airflow image. You can use this airflow image with your own modifications.

- In `master` branch of Airflow and in `v1-10-stable` branch we provide Dockerfiles and accompanying
  files that allow to build your own customized version of the Airflow Production image. To build your own image or to customize it; first clone the image and then checkout the right version which are conservative, masters and if you are adventurous then you can run `docker build`.You can add various arguments to customize it.More instructions on how to build your own image with additional dependencies (if needed) are provided in the [IMAGES.rst](IMAGES.rst#production-images) if you want to build it using `docker build` command or in [BREEZE.rst](BREEZE.rst#building-production-images) to use Breeze tool which easier interface, auto-complete, and accompanying screencast video. Breeze is a development and text environment that is developed for airflow but it also supports building production image very easily so we specify the production image flag additional extras or python version or python dev. so most of the parameters you can specify here is in command line parameters which have auto completion option. Note, that while it is possible to use master branch to build images for released Airflow versions, it might at times get broken so you should rather rely on building your own images from the v1-10-stable branch.

Airflow Summit 2020's "Production Docker Image" talk where context, architecture and customization/extension methods are [explained](https://youtu.be/wDr3Y7q2XoI).

## Beyond the Horizon

Airflow **is not** a data streaming solution. Tasks do not move data from
one to the other (though tasks can exchange metadata!). Airflow is not
in the [Spark Streaming](http://spark.apache.org/streaming/)
or [Storm](https://storm.apache.org/) space, it is more comparable to
[Oozie](http://oozie.apache.org/) or
[Azkaban](https://azkaban.github.io/).

Workflows are expected to be mostly static or slowly changing. You can think
of the structure of the tasks in your workflow as slightly more dynamic
than a database structure would be. Airflow workflows are expected to look
similar from a run to the next, this allows for clarity around
unit of work and continuity.

## Principles

- **Dynamic**:  Airflow pipelines are configuration as code (Python), allowing for dynamic pipeline generation. This allows for writing code that instantiates pipelines dynamically.
- **Extensible**:  Easily define your own operators, executors and extend the library so that it fits the level of abstraction that suits your environment.
- **Elegant**:  Airflow pipelines are lean and explicit. Parameterizing your scripts is built into the core of Airflow using the powerful **Jinja** templating engine.
- **Scalable**:  Airflow has a modular architecture and uses a message queue to orchestrate an arbitrary number of workers.

## User Interface

- **DAGs**: Overview of all DAGs in your environment.
![](/docs/img/dags.png)

- **Tree View**: Tree representation of a DAG that spans across time.
![](/docs/img/tree.png)

- **Graph View**: Visualization of a DAG's dependencies and their current status for a specific run.
![](/docs/img/graph.png)

- **Task Duration**: Total time spent on different tasks over time.
![](/docs/img/duration.png)

- **Gantt View**: Duration and overlap of a DAG.
![](/docs/img/gantt.png)

- **Code View**:  Quick way to view source code of a DAG.
![](/docs/img/code.png)


## Using hooks and Operators from "master" in Airflow 1.10

Currently stable versions of Apache Airflow are released in 1.10.* series. We are working on the
future, major version of Airflow from the 2.0.* series. It is going to be released in
in 2020. However the exact time of release depends on many factors and is yet unknown.
We have already a lot of changes in the hooks/operators/sensors for many external systems
and they are not used because they are part of the master/2.0 release.

In the Airflow 2.0 - following AIP-21 "change in import paths" all the non-core operators/hooks/sensors
of Apache Airflow have been moved to the "airflow.providers" package. This opened a possibility to
use the operators from Airflow 2.0 in Airflow 1.10 - with the constraint that those
packages can only be used in python3.6+ environment.

Therefore we decided to prepare and release backport packages that can be installed
for older Airflow versions. Those backport packages are released more frequently. Users do not
have to upgrade their Airflow version to use those packages. There are a number of changes
between Airflow 2.0 and 1.10.* - documented in [UPDATING.md](UPDATING.md). With backported
providers package users can migrate their DAGs to the new providers package incrementally
and once they convert to the new operators/sensors/hooks they can seamlessly migrate their
environments to Airflow 2.0.

More information about the status and releases of the back-ported packages are available
at [Backported providers package page](https://cwiki.apache.org/confluence/display/AIRFLOW/Backported+providers+packages+for+Airflow+1.10.*+series)

Dependencies between packages are stored in ``airflow/providers/dependencies.json``. See
[CONTRIBUTING.rst](https://github.com/apache/airflow/blob/master/CONTRIBUTING.rst#backport-providers-packages)

## Contributing

Want to help build Apache Airflow? Check out our [contributing documentation](https://github.com/apache/airflow/blob/master/CONTRIBUTING.rst).


## Who uses Airflow?

As the Apache Airflow community grows, we'd like to keep track of who is using
the platform. Please send a PR with your company name and @githubhandle
if you may.

Committers:

* Refer to [Committers](https://cwiki.apache.org/confluence/display/AIRFLOW/Committers)

Currently **officially** using Airflow:

1. [AdBOOST](https://www.adboost.sk) [[AdBOOST](https://github.com/AdBOOST)]
1. [Agari](https://github.com/agaridata) [[@r39132](https://github.com/r39132)]
1. [Airbnb](http://airbnb.io/) [[@mistercrunch](https://github.com/mistercrunch), [@artwr](https://github.com/artwr)]
1. [Airtel](https://www.airtel.in/) [[@harishbisht](https://github.com/harishbisht)]
1. [Alan](https://alan.eu) [[@charles-go](https://github.com/charles-go)]
1. [allegro.pl](http://allegro.tech/) [[@kretes](https://github.com/kretes)]
1. [AltX](https://www.getaltx.com/about) [[@pedromduarte](https://github.com/pedromduarte)]
1. [Apigee](https://apigee.com) [[@btallman](https://github.com/btallman)]
1. [ARGO Labs](http://www.argolabs.org) [[California Data Collaborative](https://github.com/California-Data-Collaborative)]
1. [Astronomer](http://www.astronomer.io) [[@schnie](https://github.com/schnie), [@andscoop](https://github.com/andscoop), [@tedmiston](https://github.com/tedmiston), [@benjamingregory](https://github.com/benjamingregory)]
1. [Auth0](https://auth0.com) [[@sicarul](https://github.com/sicarul)]
1. [Away](https://awaytravel.com) [[@trunsky](https://github.com/trunsky)]
1. [Azri Solutions](http://www.azrisolutions.com/) [[@userimack](https://github.com/userimack)]
1. [BalanceHero](http://truebalance.io/) [[@swalloow](https://github.com/swalloow)]
1. [Banco de Formaturas](https://www.bancodeformaturas.com.br) [[@guiligan](https://github.com/guiligan)]
1. [BandwidthX](http://www.bandwidthx.com) [[@dineshdsharma](https://github.com/dineshdsharma)]
1. [Bellhops](https://github.com/bellhops)
1. [BelugaDB](https://belugadb.com) [[@fabio-nukui](https://github.com/fabio-nukui) & [@joao-sallaberry](http://github.com/joao-sallaberry) & [@lucianoviola](https://github.com/lucianoviola) & [@tmatuki](https://github.com/tmatuki)]
1. [BlaBlaCar](https://www.blablacar.com) [[@puckel](https://github.com/puckel) & [@wmorin](https://github.com/wmorin)]
1. [Bloc](https://www.bloc.io) [[@dpaola2](https://github.com/dpaola2)]
1. [Blue Yonder](http://www.blue-yonder.com) [[@blue-yonder](https://github.com/blue-yonder)]
1. [BlueApron](https://www.blueapron.com) [[@jasonjho](https://github.com/jasonjho) & [@matthewdavidhauser](https://github.com/matthewdavidhauser)]
1. [Bluecore](https://www.bluecore.com) [[@JLDLaughlin](https://github.com/JLDLaughlin)]
1. [Boda Telecom Suite - CE](https://github.com/bodastage/bts-ce) [[@erssebaggala](https://github.com/erssebaggala), [@bodastage](https://github.com/bodastage)]
1. [Bodastage Solutions](http://bodastage.com) [[@erssebaggala](https://github.com/erssebaggala), [@bodastage](https://github.com/bodastage)]
1. [Bonnier Broadcasting](http://www.bonnierbroadcasting.com) [[@wileeam](https://github.com/wileeam)]
1. [BounceX](http://www.bouncex.com) [[@JoshFerge](https://github.com/JoshFerge), [@hudsonrio](https://github.com/hudsonrio), [@ronniekritou](https://github.com/ronniekritou)]
1. [California Data Collaborative](https://github.com/California-Data-Collaborative) powered by [ARGO Labs](http://www.argolabs.org)
1. [Carbonite](https://www.carbonite.com) [[@ajbosco](https://github.com/ajbosco)]
1. [Celect](http://www.celect.com) [[@superdosh](https://github.com/superdosh) & [@chadcelect](https://github.com/chadcelect)]
1. [Change.org](https://www.change.org) [[@change](https://github.com/change), [@vijaykramesh](https://github.com/vijaykramesh)]
1. [Checkr](https://checkr.com) [[@tongboh](https://github.com/tongboh)]
1. [Children's Hospital of Philadelphia Division of Genomic Diagnostics](http://www.chop.edu/centers-programs/division-genomic-diagnostics) [[@genomics-geek]](https://github.com/genomics-geek/)
1. [Cinimex DataLab](http://cinimex.ru) [[@kdubovikov](https://github.com/kdubovikov)]
1. [City of San Diego](http://sandiego.gov) [[@MrMaksimize](https://github.com/mrmaksimize), [@andrell81](https://github.com/andrell81) & [@arnaudvedy](https://github.com/arnaudvedy)]
1. [Clairvoyant](https://clairvoyantsoft.com) [@shekharv](https://github.com/shekharv)
1. [Clover Health](https://www.cloverhealth.com) [[@gwax](https://github.com/gwax) & [@vansivallab](https://github.com/vansivallab)]
1. [Chartboost](https://www.chartboost.com) [[@cgelman](https://github.com/cgelman) & [@dclubb](https://github.com/dclubb)]
1. [ContaAzul](https://www.contaazul.com) [[@bern4rdelli](https://github.com/bern4rdelli), [@renanleme](https://github.com/renanleme) & [@sabino](https://github.com/sabino)]
1. [Cotap](https://github.com/cotap/) [[@maraca](https://github.com/maraca) & [@richardchew](https://github.com/richardchew)]
1. [Craig@Work](https://www.craigatwork.com)
1. [Credit Karma](https://www.creditkarma.com/) [[@preete-dixit-ck](https://github.com/preete-dixit-ck) & [@harish-gaggar-ck](https://github.com/harish-gaggar-ck) & [@greg-finley-ck](https://github.com/greg-finley-ck)]
1. [CreditCards.com](https://www.creditcards.com/)[[@vmAggies](https://github.com/vmAggies) &  [@jay-wallaby](https://github.com/jay-wallaby)]
1. [Creditas](https://www.creditas.com.br) [[@dcassiano](https://github.com/dcassiano)]
1. [Custom Ink](https://www.customink.com/) [[@david-dalisay](https://github.com/david-dalisay), [@dmartin11](https://github.com/dmartin11) & [@mpeteuil](https://github.com/mpeteuil)]
1. [Data Reply](https://www.datareply.co.uk/) [[@kaxil](https://github.com/kaxil)]
1. [DataFox](https://www.datafox.com/) [[@sudowork](https://github.com/sudowork)]
1. [Digital First Media](http://www.digitalfirstmedia.com/) [[@duffn](https://github.com/duffn) & [@mschmo](https://github.com/mschmo) & [@seanmuth](https://github.com/seanmuth)]
1. [DocuTAP](https://www.docutap.com/) [[@jshvrsn](https://github.com/jshvrsn) & [@lhvphan](https://github.com/lhvphan) & [@cloneluke](https://github.com/cloneluke)]
1. [Dotmodus](http://dotmodus.com) [[@dannylee12](https://github.com/dannylee12)]
1. [Drivy](https://www.drivy.com) [[@AntoineAugusti](https://github.com/AntoineAugusti)]
1. [Easy Taxi](http://www.easytaxi.com/) [[@caique-lima](https://github.com/caique-lima) & [@WesleyBatista](https://github.com/WesleyBatista) & [@diraol](https://github.com/diraol)]
1. [eRevalue](https://www.datamaran.com) [[@hamedhsn](https://github.com/hamedhsn)]
1. [evo.company](https://evo.company/) [[@orhideous](https://github.com/orhideous)]
1. [FreshBooks](https://github.com/freshbooks) [[@DinoCow](https://github.com/DinoCow)]
1. [Fundera](https://fundera.com) [[@andyxhadji](https://github.com/andyxhadji)]
1. [GameWisp](https://gamewisp.com) [[@tjbiii](https://github.com/TJBIII) & [@theryanwalls](https://github.com/theryanwalls)]
1. [Gentner Lab](http://github.com/gentnerlab) [[@neuromusic](https://github.com/neuromusic)]
1. [Glassdoor](https://github.com/Glassdoor) [[@syvineckruyk](https://github.com/syvineckruyk)]
1. [Global Fashion Group](http://global-fashion-group.com) [[@GFG](https://github.com/GFG)]
1. [GovTech GDS](https://gds-gov.tech) [[@chrissng](https://github.com/chrissng) & [@datagovsg](https://github.com/datagovsg)]
1. [Grand Rounds](https://www.grandrounds.com/) [[@richddr](https://github.com/richddr), [@timz1290](https://github.com/timz1290), [@wenever](https://github.com/@wenever), & [@runongirlrunon](https://github.com/runongirlrunon)]
1. [Groupalia](http://es.groupalia.com) [[@jesusfcr](https://github.com/jesusfcr)]
1. [Groupon](https://groupon.com) [[@stevencasey](https://github.com/stevencasey)]
1. [Gusto](https://gusto.com) [[@frankhsu](https://github.com/frankhsu)]
1. [Handshake](https://joinhandshake.com/) [[@mhickman](https://github.com/mhickman)]
1. [Handy](http://www.handy.com/careers/73115?gh_jid=73115&gh_src=o5qcxn) [[@marcintustin](https://github.com/marcintustin) / [@mtustin-handy](https://github.com/mtustin-handy)]
1. [HBC Digital](http://tech.hbc.com) [[@tmccartan](https://github.com/tmccartan) & [@dmateusp](https://github.com/dmateusp)]
1. [HBO](http://www.hbo.com/)[[@yiwang](https://github.com/yiwang)]
1. [Healthjump](http://www.healthjump.com/) [[@miscbits](https://github.com/miscbits)]
1. [HelloFresh](https://www.hellofresh.com) [[@tammymendt](https://github.com/tammymendt) & [@davidsbatista](https://github.com/davidsbatista) & [@iuriinedostup](https://github.com/iuriinedostup)]
1. [Holimetrix](http://holimetrix.com/) [[@thibault-ketterer](https://github.com/thibault-ketterer)]
1. [Hootsuite](https://github.com/hootsuite)
1. [Hostnfly](https://www.hostnfly.com/) [[@CyrilLeMat](https://github.com/CyrilLeMat) & [@pierrechopin](https://github.com/pierrechopin) & [@alexisrosuel](https://github.com/alexisrosuel)]
1. [HotelQuickly](https://github.com/HotelQuickly) [[@zinuzoid](https://github.com/zinuzoid)]
1. [IFTTT](https://www.ifttt.com/) [[@apurvajoshi](https://github.com/apurvajoshi)]
1. [iHeartRadio](http://www.iheart.com/)[[@yiwang](https://github.com/yiwang)]
1. [imgix](https://www.imgix.com/) [[@dclubb](https://github.com/dclubb)]
1. [ING](http://www.ing.com/)
1. [Intercom](http://www.intercom.com/) [[@fox](https://github.com/fox) & [@paulvic](https://github.com/paulvic)]
1. [Investorise](https://investorise.com/) [[@svenvarkel](https://github.com/svenvarkel)]
1. [Jampp](https://github.com/jampp)
1. [JobTeaser](https://www.jobteaser.com) [[@stefani75](https://github.com/stefani75) &  [@knil-sama](https://github.com/knil-sama)]
1. [Kalibrr](https://www.kalibrr.com/) [[@charlesverdad](https://github.com/charlesverdad)]
1. [Karmic](https://karmiclabs.com) [[@hyw](https://github.com/hyw)]
1. [Kiwi.com](https://kiwi.com/) [[@underyx](https://github.com/underyx)]
1. [Kogan.com](https://github.com/kogan) [[@geeknam](https://github.com/geeknam)]
1. [Lemann Foundation](http://fundacaolemann.org.br) [[@fernandosjp](https://github.com/fernandosjp)]
1. [LendUp](https://www.lendup.com/) [[@lendup](https://github.com/lendup)]
1. [LetsBonus](http://www.letsbonus.com) [[@jesusfcr](https://github.com/jesusfcr) & [@OpringaoDoTurno](https://github.com/OpringaoDoTurno)]
1. [liligo](http://liligo.com/) [[@tromika](https://github.com/tromika)]
1. [LingoChamp](http://www.liulishuo.com/) [[@haitaoyao](https://github.com/haitaoyao)]
1. [Lucid](http://luc.id) [[@jbrownlucid](https://github.com/jbrownlucid) & [@kkourtchikov](https://github.com/kkourtchikov)]
1. [Lumos Labs](https://www.lumosity.com/) [[@rfroetscher](https://github.com/rfroetscher/) & [@zzztimbo](https://github.com/zzztimbo/)]
1. [Lyft](https://www.lyft.com/)[[@SaurabhBajaj](https://github.com/SaurabhBajaj)]
1. [M4U](https://www.m4u.com.br/) [[@msantino](https://github.com/msantino)]
1. [Madrone](http://madroneco.com/) [[@mbreining](https://github.com/mbreining) & [@scotthb](https://github.com/scotthb)]
1. [Markovian](https://markovian.com/) [[@al-xv](https://github.com/al-xv), [@skogsbaeck](https://github.com/skogsbaeck), [@waltherg](https://github.com/waltherg)]
1. [Mercadoni](https://www.mercadoni.com.co) [[@demorenoc](https://github.com/demorenoc)]
1. [Mercari](http://www.mercari.com/) [[@yu-iskw](https://github.com/yu-iskw)]
1. [MFG Labs](https://github.com/MfgLabs)
1. [MiNODES](https://www.minodes.com) [[@dice89](https://github.com/dice89), [@diazcelsa](https://github.com/diazcelsa)]
1. [Multiply](https://www.multiply.com) [[@nrhvyc](https://github.com/nrhvyc)]
1. [mytaxi](https://mytaxi.com) [[@mytaxi](https://github.com/mytaxi)]
1. [Nerdwallet](https://www.nerdwallet.com)
1. [New Relic](https://www.newrelic.com) [[@marcweil](https://github.com/marcweil)]
1. [Newzoo](https://www.newzoo.com) [[@newzoo-nexus](https://github.com/newzoo-nexus)]
1. [Nextdoor](https://nextdoor.com) [[@SivaPandeti](https://github.com/SivaPandeti), [@zshapiro](https://github.com/zshapiro) & [@jthomas123](https://github.com/jthomas123)]
1. [OdysseyPrime](https://www.goprime.io/) [[@davideberdin](https://github.com/davideberdin)]
1. [OfferUp](https://offerupnow.com)
1. [OneFineStay](https://www.onefinestay.com) [[@slangwald](https://github.com/slangwald)]
1. [Open Knowledge International](https://okfn.org) [@vitorbaptista](https://github.com/vitorbaptista)
1. [Overstock](https://www.github.com/overstock) [[@mhousley](https://github.com/mhousley) & [@mct0006](https://github.com/mct0006)]
1. [Pandora Media](https://www.pandora.com/) [[@Acehaidrey](https://github.com/Acehaidrey) & [@wolfier](https://github.com/wolfier)]
1. [PAYMILL](https://www.paymill.com/) [[@paymill](https://github.com/paymill) & [@matthiashuschle](https://github.com/matthiashuschle)]
1. [PayPal](https://www.paypal.com/) [[@r39132](https://github.com/r39132) & [@jhsenjaliya](https://github.com/jhsenjaliya)]
1. [Pernod-Ricard](https://www.pernod-ricard.com/) [[@romain-nio](https://github.com/romain-nio)]
1. [Plaid](https://www.plaid.com/) [[@plaid](https://github.com/plaid), [@AustinBGibbons](https://github.com/AustinBGibbons) & [@jeeyoungk](https://github.com/jeeyoungk)]
1. [Playbuzz](https://www.playbuzz.com/) [[@clintonboys](https://github.com/clintonboys) & [@dbn](https://github.com/dbn)]
1. [PMC](https://pmc.com/) [[@andrewm4894](https://github.com/andrewm4894)]
1. [Postmates](http://www.postmates.com) [[@syeoryn](https://github.com/syeoryn)]
1. [Pronto Tools](http://www.prontotools.io/) [[@zkan](https://github.com/zkan) & [@mesodiar](https://github.com/mesodiar)]
1. [PubNub](https://pubnub.com) [[@jzucker2](https://github.com/jzucker2)]
1. [Qplum](https://qplum.co) [[@manti](https://github.com/manti)]
1. [Quantopian](https://www.quantopian.com/) [[@eronarn](http://github.com/eronarn)]
1. [Qubole](https://qubole.com) [[@msumit](https://github.com/msumit)]
1. [Quizlet](https://quizlet.com) [[@quizlet](https://github.com/quizlet)]
1. [Quora](https://www.quora.com/)
1. [REA Group](https://www.rea-group.com/)
1. [Reddit](https://www.reddit.com/) [[@reddit](https://github.com/reddit/)]
1. [Robinhood](https://robinhood.com) [[@vineet-rh](https://github.com/vineet-rh)]
1. [Scaleway](https://scaleway.com) [[@kdeldycke](https://github.com/kdeldycke)]
1. [Sense360](https://github.com/Sense360) [[@kamilmroczek](https://github.com/KamilMroczek)]
1. [Shopkick](https://shopkick.com/) [[@shopkick](https://github.com/shopkick)]
1. [Sidecar](https://hello.getsidecar.com/) [[@getsidecar](https://github.com/getsidecar)]
1. [SimilarWeb](https://www.similarweb.com/) [[@similarweb](https://github.com/similarweb)]
1. [SmartNews](https://www.smartnews.com/) [[@takus](https://github.com/takus)]
1. [SocialCops](https://www.socialcops.com/) [[@vinayak-mehta](https://github.com/vinayak-mehta) & [@sharky93](https://github.com/sharky93)]
1. [Spotahome](https://www.spotahome.com/) [[@spotahome](https://github.com/spotahome)]
1. [Spotify](https://github.com/spotify) [[@znichols](https://github.com/znichols)]
1. [Stackspace](https://beta.stackspace.io/)
1. [Stripe](https://stripe.com) [[@jbalogh](https://github.com/jbalogh)]
1. [Tails.com](https://tails.com/) [[@alanmcruickshank](https://github.com/alanmcruickshank)]
1. [Thinking Machines](https://thinkingmachin.es) [[@marksteve](https://github.com/marksteve)]
1. [Thinknear](https://www.thinknear.com/) [[@d3cay1](https://github.com/d3cay1), [@ccson](https://github.com/ccson), & [@ababian](https://github.com/ababian)]
1. [Thumbtack](https://www.thumbtack.com/) [[@natekupp](https://github.com/natekupp)]
1. [Tictail](https://tictail.com/)
1. [Tile](https://tile.com/) [[@ranjanmanish](https://github.com/ranjanmanish)]
1. [Tokopedia](https://www.tokopedia.com/) [@topedmaria](https://github.com/topedmaria)
1. [Twine Labs](https://www.twinelabs.com/) [[@ivorpeles](https://github.com/ivorpeles)]
1. [Twitter](https://www.twitter.com/) [[@aoen](https://github.com/aoen)]
1. [T2 Systems](http://t2systems.com) [[@unclaimedpants](https://github.com/unclaimedpants)]
1. [Ubisoft](https://www.ubisoft.com/) [[@Walkoss](https://github.com/Walkoss)]
1. [United Airlines](https://www.united.com/) [[@ilopezfr](https://github.com/ilopezfr)]
1. [Upsight](https://www.upsight.com) [[@dhuang](https://github.com/dhuang)]
1. [Vente-Exclusive.com](http://www.vente-exclusive.com/) [[@alexvanboxel](https://github.com/alexvanboxel)]
1. [Vevo](https://www.vevo.com/) [[@csetiawan](https://github.com/csetiawan) & [@jerrygillespie](https://github.com/jerrygillespie)]
1. [Vnomics](https://github.com/vnomics) [[@lpalum](https://github.com/lpalum)]
1. [WePay](http://www.wepay.com) [[@criccomini](https://github.com/criccomini) & [@mtagle](https://github.com/mtagle)]
1. [WeTransfer](https://github.com/WeTransfer) [[@jochem](https://github.com/jochem)]
1. [Whistle Labs](http://www.whistle.com) [[@ananya77041](https://github.com/ananya77041)]
1. [WiseBanyan](https://wisebanyan.com/)
1. [Wooga](https://www.wooga.com/)
1. [Xero](https://www.xero.com/) [[@yan9yu](https://github.com/yan9yu)]
1. [Xoom](https://www.xoom.com/)
1. [Yahoo!](https://www.yahoo.com/)
1. [Yieldr](https://www.yieldr.com/) [[@ggeorgiadis](https://github.com/ggeorgiadis)]
1. [Zapier](https://www.zapier.com) [[@drknexus](https://github.com/drknexus) & [@statwonk](https://github.com/statwonk)]
1. [Zego](https://www.zego.com/) [[@ruimffl](https://github.com/ruimffl)]
1. [Zendesk](https://www.github.com/zendesk)
1. [Zenly](https://zen.ly) [[@cerisier](https://github.com/cerisier) & [@jbdalido](https://github.com/jbdalido)]
1. [Zymergen](https://www.zymergen.com/)
1. [99](https://99taxis.com) [[@fbenevides](https://github.com/fbenevides), [@gustavoamigo](https://github.com/gustavoamigo) & [@mmmaia](https://github.com/mmmaia)]

## Who Maintains Apache Airflow?

Airflow is the work of the [community](https://github.com/apache/airflow/graphs/contributors),
but the [core committers/maintainers](https://people.apache.org/committers-by-project.html#airflow)
are responsible for reviewing and merging PRs as well as steering conversation around new feature requests.
If you would like to become a maintainer, please review the Apache Airflow
[committer requirements](https://cwiki.apache.org/confluence/display/AIRFLOW/Committers).

## Can I use the Apache Airflow logo in my presentation?

Yes! Be sure to abide by the Apache Foundation [trademark policies](https://www.apache.org/foundation/marks/#books) and the Apache Airflow [Brandbook](https://cwiki.apache.org/confluence/display/AIRFLOW/Brandbook). The most up to date logos are found in [this repo](/docs/img/logos) and on the Apache Software Foundation [website](https://www.apache.org/logos/about.html).

## Airflow merchandise

If you would love to have Apache Airflow stickers, t-shirt etc. then check out
[Redbubble Shop](https://www.redbubble.com/i/sticker/Apache-Airflow-by-comdev/40497530.EJUG5).

## Links


- [Documentation](https://airflow.apache.org/)
- [Chat](https://s.apache.org/airflow-slack)
- [More](https://cwiki.apache.org/confluence/display/AIRFLOW/Airflow+Links)