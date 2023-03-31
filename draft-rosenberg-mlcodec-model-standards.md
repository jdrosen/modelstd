---
title: Standardizing Machine Learning Models at the IETF
abbrev: Model Standards
docname: draft-rosenberg-mlcodec-model-standards
date: 2023-04-02

# stand_alone: true

ipr: trust200902
area: Applications
wg: MLCodec
kw: Internet-Draft
cat: std

coding: us-ascii
pi:    # can use array (if all yes) or hash here
#  - toc
#  - sortrefs
#  - symrefs
  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:
      -
        ins: J. Rosenberg
        name: Jonathan Rosenberg
        org: Five9
        email: jdrosen@jdrosen.net
      


normative:
    RFC6716:
    I-D.valin-opus-dred:


informative:

--- abstract

This document describes a process for standardizing machine learning models used within an Internet protocol. Machine learning models are complex mathematical structures, not well suited for embedding in a document. In addition, these models often require updates on a timetable faster than the normal IETF standardization process permits. The process define here allows models to be updated through working group consensus, and then have those updates propagate in real-time to implementations, all while maintaining backwards compatibility. 

--- middle

# Introduction  

Machine learning models are now beginning to appear in the design of protocols. For example, {{I-D.valin-opus-dred}} specifies an extension to the Opus codec {{RFC6716}}. This extension includes the usage of a Deep Neural Network (DNN) in the encoder, producing a bitstream which is sent to the decoder and processed by a second DNN. 

The usage of machine learning models presents several difficulties when using them within IETF standards.

Firstly - these models cannot be defined using English prose. Models are defined by a combination of a model architecture, along with a set of parameters. While it is possible to describe the model architecture in ENglish prose and accompanying diagrams, the model parameters cannot be expressed that way. Model parameters are an array of numbers, often measuring in the millions or more of parameters. It is not practical nor useful to print them out and list them in an RFC. 

Secondly - machine learning models are continuously updated in practical applications. These updates happen when the model is retrained based on additional data, or a new architecture is utilized and retrained on existing or new data. These updates are usually for the purposes of increased accuracy or reduced computational complexity when perfomring an inference. When utilizing a machine learning model in an Internet protocol, we do not want to lose the capability to make updates to the model. The existing IETF standards process is ill suited to deliver these updates. First, since the model updates are just changes in parameters and architecture, they are not a document which can be reviewed by the IESG or the community. Secondly, the standards process at IETF usually takes months or years, and this is well beyond the desired timetables for model updates (days or weeks at worst). 


# Solution Overview


## Standardizing a Format

The IETF would stanardize a format for representing machine learning models. There are already many of them in widespread usage - JSON structures, YAML files, and so on - all exist. IETF would pick one that is ideally language agnostic, and standardize it through the normal IETF standards process.


## Publishing and Updating Models

With this format standardized, the IETF would enable publication of models in this format through github. The IETF would have its own organization on github, which are maintained and supported by the secretariat. Working groups which seek to publish standards making use of a machine learning model, would have repositories created in github by the secretariat. These repositories would be created at the time of chartering of a working group that requires such a repository. Additionally, the working group chairs could ask the secretariat at any time to create a new repositoriy. Repository names would exactly match the working group name which is using them. The repository would have permissions assigned, such that only working group chairs are allowed to approve merge requests within the repostiory. 

If a particular specification within the working group requires models, that document would have a section that provides a name for the model(s) it requires. When a document is adopted as a working group item, the document authors would also submit a merge request that contain the initial version of the model. The model itself is versioned, and a particular version is immutable. So, for example, if a model was called "opus-param-encoder", the initial version would be "opus-param-encoder-00". Once that version appears in the repository, it cannot change. The working group chairs will never approve merge requests which modify an existing model. The only permitted MRs are those which add a new version.

Model updates are performed via the working group consensus process. A model update would occur in a feature branch pulled by the authors of the document containing those models. Implementors within the working group could test and confirm that they are happy with the new model update. When the working group is ready to do an update to the model, the working group chair would obtain consensus through the normal IETF means. If the chairs determine that consensus has been reached, the authors submit a merge request for that branch. The chairs approve the merge request. As noted above, the chairs are responsible for making sure the merge request does not modify the prior version, and the new version has an increased version number. 

Model updates can be done independently of document updates. It is not required to update the document, to update the model. 

When the document itself goes to RFC, the final document would however point to the version of the model which was considered "good enough" to be used as part of the completed standard. So, continuing the example above, a document referencing "opus-param-encoder" would have, in its published RFC form, a statement along the lines of "This specification is used with opus-param-encoder-06 or better". Note the usage of the words "or better", which will allow model updates after the RFC itself is published.

Model updates can continue during the RFC approval process and after the RFC itself is published. These are always done via working group consensus as described above. The working group is responsible for making sure that the updated models do not break backwards compatibility. The working group chairs are accountable for making sure that this has been validated by the working group, as a result of real-world interoperability testing which is demonstrated to the working group. 

## Run-Time Consumption of Models

Implementations of the protocols would be able to fetch the model at run-time, and make use of whatever version the implementation desires. New versions can be fetched when an implementation starts up, or they can be fetched periodically, or they can be fetched dynamically as a part of message exchange. These details would be defined as part of the protocol specification, depending on the needs of the protocol. 

In cases where the protocol requires multiple implementations to have the same version, the protocol would need to specify techniques for exchanging supported version numbers and/or exchanging required versions that trigger other participants to download the version of the model. 

It is expected that, a common technique will be "triggered download". In this model, one implementation will send a message based on a particular version of a model. This will be received by a different implementation, build by a different vendor. The message indicates the model version that is required to process the message. The second implementation will download the model and then use it to process the message. This process can take several seconds (models can be megabytes or more), and thus would typically be done at a point in time where the user can wait. For example, if used in a meeting product, the download could happen when the user attempts to join the meeting. It is common practice for meeting joins to require users to download the latest version of a meeting client. Downloading a model is similar. 

A second mode of operation would be "download on launch". In this use case, when a user starts their implementation, it downloads the most recent version of the model and uses it. This introduces a possibility that two different implementations do not have the latest. If a specification wants to supprot this mode, the protocol itself would need to allow implementations to exchange information on what versions of the model they have, and agree on a common one. To ensure interoperability, all implementations would need to at least have the version of the model that was current as of publication of the RFC. As noted above, that version is documented in the RFC itself. 

A third mode of operation is "periodic download". This mode is similar to download on launch. In this mode, the implementation will periodically (say, once a month), check for updates and download the latest model. The underlying protocol will need to support the same negotiation as described above for download at launch.


## Distribution of Models

As discussed above, models produced by the IETF will be need to be accessed by implementations during the run-time operation. To ensure this can be done, the IETF will contract with a commercial CDN provider, and provide highly reliable, low latency, global distribution of these models. 

The Github pipeline associated with a merge to main, would trigger publication of the new model into the CDN. This would happen automatically. This means that new versions of a model would be available to running applications within minutes of publication to the CDN. 

The IETF would have a well-defined naming structure for models. For example:

~~~~~~~~~~~
https://cdn.ietf.org/wg/mlcodec/models/opus-param-encoder-05.yaml
~~~~~~~~~~~


## Model Catalog

To allow applications to know whether there is a new version of a model or not, each repository would have a catalog file. This catalog file would also be a well-known structure, defined by an RFC. This structure would basically be list of model versions, and for each version, a date and status. The date represents the date it was published. The status would indicate whether this version is the most recent, obsoleted, or active (where active means it can still be used but is not the most recent). This would allow an implementation to fetch the catalog at any time, and learn whether a more recent version of the model is available.

The catalog would also be distributed via CDN, and also have a well-defined URL so that it can always be retrieved. 


# Discussion Points

This document defines a bunch of processes which are very new and different to how IETF operates. Certainly, there are many points of consideration.

## Github vs. run git

An alternative to utilizing github is to actually have the IETF run and operate its own git instance. THere are tradeoffs here.

By running its own instance, IETF would eliminate the challenges associated with selection of a third party vendor for services. Vendor selection at IETF is a challenging process. Though this document assumes github, certainly there would be proponents for gitlab or some other vendor. Would IETF need to produce an RFC that defines requirements, and then go through a formal vendor selection process, including RFP and eventual decision that is ratified by the community? That could be a really long, multi-year process. If IETF just runs git itself, that is avoided.

On the other hand, IETF is not in the business of being a SaaS vendor and running highly available software for consumption by the Internet community. Though IETF does do it (see - datatracker), this ups the bar further. Should IETF really be in this business? It seems better to leave it to third parties who are experts in doing this. Furthermore, though we could go through a lengthy vendor selection process - we could alternatively just use github and move on. In practice, a huge amount of IETF specifications are already on github. Lets just formalize that and move on. 

## Commercial CDN

Perhaps the most contentious piece of this proposal, is the suggestion that IETF become accountable for the distribution of models by utilizing a commercial CDN vendor. The alternative is to have implementations be responsible for that. 

The main rationale for having IETF do it - is that it will enable interoperability between implementations that make use of the "triggered download" technique described above. We need a way for the second vendor to obtain a copy of any particular model at any time. Even if each vendor tries to keep their copy fresh, there is a risk of a race condition wherein vendor 1 has downloaded the latest model, sends a message to vendor 2's implementation, but vendor 2 has not yet pulled that model from the Github repository. That will cause an interop failure. By making the IETF the global source of truth for the model, this race is eliminated.

There are however significant drawbacks to having the IETF take this responsibility. The first is cost. CDN vendors charge for downloads, and that cost would be incurred by the IETF. This means the IETF could also be subject to uncapped costs, should an attacker just repeatedly download files to cause harm. Even ignoring this, the IETF would need to pay for the normal runtime use of the CDN, and that is not something it may be able to budget for. The second drawback is the requirement for a vendor selection, which is likely to be far more contentious than selecting a repository.

If IETF does not provide distribution of the models, all protocols would need to support the case where implementations do not share the same version of the model. This means the "triggered download" use case would not really be possible. 

The author leans to NOT having the IETF be in the business of distribution, but this is included in the document to facilitate community discussion.

## Lack of Review

This document proposes that model updates take place solely through working group consensus. This means there is no IESG review, no expert review, no IETF community review. The number of "eyeballs" that are on model updates will, consequently, be much smaller. Indeed, it is not really possible to even validate that the new model is good except for actually using it in an implementation. This means that implementors are critical to the process. There is no document that even a working group member, who is not an implementor, can review. One might argue that this is not enough checks and balances. 

On the other hand, there is simply no alternative. Even if models were pasted into an RFC as a long array of numbers, they would not be something that can be reviewed or validated outside of implementors anyway. Secondly, the dynamic nature of this system brings down the costs of a bad model. Implementations can just go back to a different one. The catalog structure defined above would even enable working group consensus to deactivate a bad model, and for that to propagate to implementations globally almost instantaneously. 

This mirrors modern software practice. If you can deploy frequently, you can test less prior to deployment. In the past, when IETF RFCs were coded into implementations that might never get updated, correctness was more important than anything else. The extensive review process at the IETF provides better correctness (though, there are of course errata, so it is no guarantee), at the cost of speed. With modern software that can update more frequently, this is less important. 










