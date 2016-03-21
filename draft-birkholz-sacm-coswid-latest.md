---
title: Concise Software Identifiers
abbrev: COSWID
docname: draft-birkholz-sacm-coswid-latest
stand_alone: true
ipr: trust200902
area: Security
wg: SACM Working Group
kw: Internet-Draft
cat: std
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- ins: J. Fitzgerald-McKay
  name: Jessica Fitzgerald-McKay
  org: Department of Defense
  email: jmfitz2@nsa.gov
  street: 9800 Savage Road
  city: Ft. Meade
  region: Maryland
  country: USA
- ins: C. Schmidt
  name: Charles Schmidt
  org: The MITRE Corporation
  email: cmschmidt@mitre.org
  street: 202 Burlington Road
  city: Bedford
  region: Maryland
  code: '01730'
  country: USA
- ins: D. Waltermire
  name: David Waltermire
  org: National Institute of Standards and Technology
  abbrev: NIST
  email: david.waltermire@nist.gov
  street: 100 Bureau Drive
  city: Gaithersburg
  region: Maryland
  code: '20877'
  country: USA

normative:
  RFC7049: cbor
  X.1520:
    title: "ITU-T X.1520 (01/2014)"
  SAM:
    title: >
      Information technology - Software asset management - Part 5: Overview and vocabulary
    date: 2013-11-15
    seriesinfo:
      ISO/IEC: 19770-5:2013
  SWID:
    title: >
      Information technology - Software asset management - Part 2: Software identification tag'
    date: 2015-10-01
    seriesinfo:
      ISO/IEC: 19770-2:2015
  I-D.ietf-cose-msg: cose-msg

informative:
  I-D.greevenbosch-appsawg-cbor-cddl: cddl
  I-D-birkholz-tuda:
    -: tuda
    title: Time-Based Uni-Directional Attestation
    date: 2016-03-21
    seriesinfo:
      Internet-Draft: draft-birkholz-tuda-01
    author:
      - ins: A. Fuchs
        name: Andreas Fuchs
      - ins: H. Birkholz
        name: Henk Birkholz
      - ins: I. McDonald
        name: Ira E McDonald
      - ins: C. Bormann
        name: Carsten Bormann

--- abstract 

This document defines a concise representation of ISO 19770-2:2015 Software Identifiers (SWID tags) that is interoperable with the XML schema definition of ISO 19770-2:2015. 

--- middle

# Introduction

SWID tags have several use-applications including but not limited to:

* Software Inventory Management, a part of the Software Asset Management {{SAM}} process, which requires an accurate list of discernible deployed software instances.

* Vulnerability Assessment, which requires a semantic link between standardized vulnerability descriptions and {{X.1520}} IT-assets.

* Remote Attestation, which requires a link between golden (well-known) measurements and software instances {{-tuda}}.

SWID tags, as defined in ISO-19770-2:2015 {{SWID}}, provide a standardized format for a record that identifies and describes a specific release of a software product. Different software products, and even different releases of a particular software product, each have a different SWID tag record associated with them. In addition to defining the format of these records, ISO-19770-2:2015 defines requirements concerning the SWID tag lifecycle. Specifically, when a software product is installed on an endpoint, that product's SWID tag is also installed. Likewise, when the product is uninstalled or replaced, the SWID tag is deleted or replaced, as appropriate. As a result, ISO-19770-2:2015 describes a system wherein there is a correspondence between the set of installed software products on an endpoint, and the presence on that endpoint of the SWID tags corresponding to those products.

SWID tags are meant to be flexible and able to express a broad set of metadata about a software product. Moreover, there are multiple types of SWID tags, each providing different types of information. For example, a "corpus tag" is used to describe an application's installation image on an installation media, while a "patch tag" is meant to describe a patch that modifies some other application. While there are very few required fields in SWID tags, there are many optional fields that support different uses of these different types of tags. While a SWID tag that consisted only of required fields could be a few hundred bytes in size, a tag containing many of the optional fields could be many orders of magnitude larger.

This document defines a more concise representation of SWID tags in the Concise Binary Object Representation (CBOR) {{-cbor}}.  This is described via the CBOR Data Definition Language (CDDL) {{-cddl}}.  The resulting Concise SWID data definition is interoperable with the XML schema definition of ISO-19770-2:2015 {{SWID}}. The vocabulary, i.e., the CDDL names of the types and members used in the Concise SWID data definition, is mapped to more concise labels represented as small integers. The names used in the CDDL and the mapping to the CBOR representation using integer labels is based on the vocabulary of the XML attribute and element names defined in ISO-19770-2:2015.

Real-world instances of SWID tags can be fairly large, and the communication of SWID tags in use-applications such as those described earlier can cause a large amount of data to be transported. This can be larger than acceptable for constrained devices and networks. Concise SWID tags significantly reduce the amount of data transported as compared to a typical SWID tag. This reduction is enable through the use of CBOR, which maps human-readable labels of that content to more concise integer labels (indices). This allows SWID tags to be part of an enterprise security solution for a wider range of endpoints and environments.

# Concise SWID data definition

The following is a CDDL representation of the ISO-19770-2:2015 {{SWID}} XML schema definition of SWID tags. This representation includes all SWID tag fields and thus supports all SWID tag use cases. The CamelCase notation used in the XML schema definition is changed to hyphen-separated notation (e.g. ResourceCollection is named resource-collection in the Concise SWID data definition). The human-readable names of members are mapped to integer indices via a block of rules at the bottom of the Concise SWID data definition. The 48 character strings of the SWID vocabulary that would have to be stored or transported in full if using the original vocabulary are replaced.

~~~ CDDL

software-identity = {
  global-attr,
  ? content,
  ? corpus,
  ? patch,
  ? media,
  name,
  ? supplemental,
  tag-id,
  ? tag-version,
  ? version,
  ? version-scheme,
}

NMTOKEN = text            ; .regexp to add some validation?
NMTOKENS = [* NMTOKEN]

any-attr = text
any-element = any

date-time = time
any-uri = uri

global-attr = (
  * (text => any-attr),
  ? lang,
)

meta-type = (
  global-attr,
  * (text => any-attr),
)

meta-element = [
  global-attr,
  * (text => any-attr),
]

resource-collection = (
  global-attr,
  * (directory-entry // file-entry // process-entry // resource-entry)
)

file = {
  filesystem-item,
  ? size,
  ? version,
  * (text => any-attr),
}

filesystem-item = (
  meta-type,
  ? key,
  ? location,
  name,
  ? root,
)

directory = {
  filesystem-item,
  path-elements,
}

process = {
  global-attr,
  name,
  ? pid,
}

resource = {
  global-attr,
  type,
}

entity = {
  global-attr,
  meta-elements,
  name,
  ? reg-id,
  role,
  ? thumbprint,
}

evidence = {
  global-attr,
  resource-collection,
  ? date,
  ? device-id,
}

link = {
  global-attr,
  ? artifact,
  href,
  ? media,
  ? ownership,
  rel,
  ? type,
  ? use,
}

software-meta = {
  global-attr,
  ? activation-status,
  ? channel-type,
  ? colloquial-version,
  ? description,
  ? edition,
  ? entitlement-data-required,
  ? entitlement-key,
  ? generator,
  ? persistent-id,
  ? product,
  ? product-family,
  ? revision,
  ? summary,
  ? unspsc-code,
  ? unspsc-version,
}

payload = {
  global-attr,
  resource-collection,
}

tag-id = (0: text)
name = (1: text)
content = (
  2: [ * entity / evidence / link / software-meta /
       payload / any-element ]
)
corpus = (3: bool)
patch = (4: bool)
media = (5: text)
supplemental = (6: bool)
tag-version = (7: integer)
version = (8: text)
version-scheme = (9: NMTOKEN)
lang = (10: text)
directory-entry = (11: directory)
file-entry = (12: file)
process-entry = (13: process)
resource-entry = (14: resource)
size = (15: integer)
key = (16: bool)
location = (17: text)
root = (18: text)
path-elements = (19: ([ * (directory / file) ]))
pid = (20: integer)
type = (21: text)
meta-elements = (22: ([ * meta-element ]))
reg-id = (23: any-uri)
role = (24: NMTOKENS)
thumbprint = (25: text)
date = (26: date-time)
device-id = (27: text)
artifact = (28: text)
href = (29: any-uri)
ownership = (30: ("shared" / "private" / "abandon"))
rel = (31: NMTOKEN)
use = (32: ("optional" / "required" / "recommended"))
activation-status = (33: text)
channel-type = (34: text)
colloquial-version = (35: text)
description = (36: text)
edition = (37: text)
entitlement-data-required = (38: bool)
entitlement-key = (39: text)
generator = (40: text)
persistent-id = (41: text)
product = (42: text)
product-family = (43: text)
revision = (44: text)
summary = (45: text)
unspsc-code = (46: text)
unspsc-version = (47: text)

~~~

# COSE signatures for Concise SWID tags

SWID tags, as defined in the ISO-19770-2:2015 XML schema, can include cryptographic signatures to protect the integrity of the SWID tag. In general, tags are signed by the tag creator (typically, although not exclusively, the vendor of the software product that the SWID tag identifies). Cryptographic signatures can make any modification of the tag detectable, which is especially important if the integrity of the tag is important, such as when the tag is providing golden measurements for files.

The ISO-19770-2:2015 XML schema uses XML DSIG to support cryptographic signatures. Concise SWID tags require a different signature scheme than this. COSE provides the required mechanism {{-cose-msg}}, which will be addressed in a future iteration of this draft and most likely result in additional attributes to be included in the general Concise SWID data definition, e.g. signature-type (“compat”, “cose”, etc.). Note that, by their natures, cryptographic signatures will not remain valid if a SWID tag is translated from one representation to another.

#  IANA considerations

This document will include requests to IANA: Integer indices for SWID content attributes and information elements.

#  Security Considerations

SWID tags contain public information about software products and, as
such, do not need to be protected against disclosure on an endpoint.
Similarly, SWID tags are intended to be easily discoverable by
applications and users on an endpoint in order to make it easy to
identify and collect all of an endpoint's SWID tags.  As such, any
security considerations regarding SWID tags focus on the application
of SWID tags to address security challenges, and the possible
disclosure of the results of those applications.

A signed SWID tag whose signature is intact can be relied upon to be
unchanged since it was signed.  If the SWID tag was created by the
software author, this generally means that it has undergone no change
since the software application with which the tag is associated was
installed.  By implication, this means that the signed tag reflects
the software author's understanding of the details of that software
product.  This can be useful assurance when the information in the
tag needs to be trusted, such as when the tag is being used to convey
golden measurements.  By contrast, the data contained in unsigned
tags cannot be trusted to be unmodified.

SWID tags are designed to be easily added and removed from an
endpoint along with the installation or removal of software products.
On endpoints where addition or removal of software products is
tightly controlled, the addition or removal of SWID tags can be
similarly controlled.  On more open systems, where many users can
manage the software inventory, SWID tags may be easier to add or
remove.  On such systems, it may be possible to add or remove SWID
tags in a way that does not reflect the actual presence or absence of
corresponding software products.  Similarly, not all software
products automatically install SWID tags, so products may be present
on an endpoint without providing a corresponding SWID tag.  As such,
any collection of SWID tags cannot automatically be assumed to
represent either a complete or fully accurate representation of the
software inventory of the endpoint.  However, especially on devices
that more strictly control the ability to add or remove applications,
SWID tags are an easy way to provide an preliminary understanding of
that endpoint's software inventory.

Any report of an endpoint's SWID tag collection provides
information about the software inventory of that endpoint.  If such a
report is exposed to an attacker, this can tell them which software
products and versions thereof are present on the endpoint.  By
examining this list, the attacker might learn of the presence of
applications that are vulnerable to certain types of attacks.  As
noted earlier, SWID tags are designed to be easily discoverable by an
endpoint, but this does not present a significant risk since an
attacker would already need to have access to the endpoint to view
that information.  However, when the endpoint transmits its software
inventory to another party, or that inventory is stored on a server
for later analysis, this can potentially expose this information to
attackers who do not yet have access to the endpoint.  As such, it is
important to protect the confidentiality of SWID tag information that
has been collected from an endpoint, not because those tags
individually contain sensitive information, but because the
collection of SWID tags and their association with an endpoint
reveals information about that endpoint's attack surface.

Finally, both the ISO-19770-2:2015 XML schema definition and the
Concise SWID data definition allow for the construction of "infinite"
SWID tags or SWID tags that contain malicious content with the intend
if creating non-deterministic states during validation or processing of SWID tags. While software product vendors are unlikely to do this, SWID tags can be created by any party and the SWID tags collected from an endpoint could contain a mixture of vendor and non-vendor created tags. For this reason, tools that consume SWID tags ought to treat the tag contents as potentially malicious and should employ input sanitizing on the tags they ingest.

#  Acknowledgements

#  Change Log

First version -00

# Contributors

--- back

<!--  LocalWords:  SWID verifier TPM filesystem discoverable
 -->