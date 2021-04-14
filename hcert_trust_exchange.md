# Electronic Health Certificate Trust Exchange Specification

Version 0.01-draft, 2021-04-14


## Abstract

This document specifies a basic service model and data exchange formats for an initial infrastructure for trust information exchange to support validation of electronic health certificates.

### Version History

| version | status | Comments |
|----------|----------|----------|
| 0.01  | draft | first version |

## Terminology

Organisations adopting this specification for issuing health certificates are called Issuers and organisations accepting health certificates as proof of health status are called Verifiers. Together, these are called Participants. Issuers are registered under a country and Verifiers are assumed to be operating within a country infrastructure. These countries are called Participating Countries, or just Countries. Some aspects in this document must be coordinated between the Participants and Participant Countries, such as the management of a namespace and the distribution of cryptographic keys. It is assumed that a party, hereafter referred to as the Secretariat, carries out these tasks. The health certificate format of this specification is called the Electronic Health Certificate, hereafter referred to as the HCERT.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 ([RFC2119](https://tools.ietf.org/html/rfc2119), [RFC8174](https://tools.ietf.org/html/rfc8174)) when, and only when, they appear in all capitals, as shown here.

### Versioning Policy

Versions of this specification follow [semantic versioning](semver.org) and consist of three different integers describing the _major_, _minor_ and _edition_ version. A change in the _major_ version is an update that includes material changes affecting the decoding of the HCERT or the validation of it. An update of the _minor_ version is a feature or maintenance update that maintains backward compatibility with previous versions.

In addition, there is an _edition_ version number used for publishing updates to the document itself which has no effect on the HCERT, such as correcting spelling, providing clarifications or addressing ambiguities, et cetera. Hence, the edition number is not indicated in the HCERT. The version numbers are expressed in the title page of the document using a _major.minor.edition_ format, where the three parts are separated by decimal dots.

## Certificate trust model

HCERTs are verified using a Document Signer Certificate (DSC) that holds the public key of the document signer that issued the HCERT. A DSC MAY be self issued or signed by a Certificate Signing Certificate Authority (CSCA).

Each Participating Country is REQUIRED to provide a list of valid Document Signing Certificates (DSCs), and keep these lists current. This list is called the DSC list. All public keys within certificates present on the DSC list is considered valid for validating signatures of HCERTs from that country, regardless of other information in the certificate or whether the DSC is self signed or signed by a CSCA.

Each Participating Country MAY provide a list of one or more CSCA certificates used to sign certificates on the country DSC list. Participating countries and their verifiers MAY use the CSCA certificates as an instrument to validate certificates on the DSC list but MUST NOT require a DSC to be signed by a CSCA.

### Certificate profile

Requirements on certificate content of DSC and CSCA certificates are defined in the HCERT specification (ref).

## Certificate exchange model

### Basic concept

DSC and CSCA certificates are exchanged between each Participating Country based on information provided by the Secretariat.

All trust information is provided openly and freely without restrictions. Each Participating Country downloading certificates MUST have the means to validate and authenticate downloaded certificates, but the Participating Country providing the certificates MUST NOT require authentication of the downloading client.

The Secretariat maintains a list of Participating Countries and the following information for each country:

- URL:s to downloadable certificate sets (DSC and CSCA certificates)
- Information about how to authenticate certificates from each Country

This information can be downloaded and authenticated by other Participating Countries using the Secretariat's master URL and master key. This master key and master URL is the only information that is required to bootstrap trust in all HCERTs issued by all Participating Countries.


### Data formats

This specification defines two supported formats for providing a list of certificates. DSC and CSCA certificates MUST be provided in separate lists made available using separate URLs. The two supported formats are:

- A text document containing an unsigned list of PEM formatted certificates (ref)
- A signed set of JSON web keys ([RFC7517](https://tools.ietf.org/html/rfc7517))

All Issuing Countries MUST provide a list of PEM formatted certificates, and MAY provide a signed set of JWKs for each type of certificates they publish (DSC and CSCA certificates).

Each data format and each certificate type MUST be published through a separate URL.

### Data integrity and data origin authentication

Basic data integrity and data origin authentication SHALL be provided by using TLS version 1.2, or higher, with strong cipher suites. The TLS certificate used by each Participating Country SHALL be provided by the Secretariat via the master URL.

The key used to verify any present signed set of JWKs SHALL be provided by the Secretariat via the master URL.

## Secretariat trust information exchange service

The Secretariat offers information about Participating Countries via the master URL. The following information is provided for each Country:

- URL for downloading a list of PEM formatted DSCs
- URL for downloading a list of PEM formatted CSCA certificates
- URL for downloading a signed set of JWKs containing DSCs (Optional)
- URL for downloading a signed set of JWKs containing CSCA certificates (Optional)
- TLS Certificate
- Certificate for verifying signed JWKs (Optional)

The manner in which the Secretariat obtains this information from each Participating Country is outside the scope of this specification.


## Security considerations

This specification builds on Transport Layer Security (TLS) as the fundamental layer of protection with regard to the authenticity, integrity and recency of data and to prevent data substitution and replay attacks.

In cases where data is obtained using a signed set of JWKs, verifiers MAY ignore the TLS protection and rather depend on protection on the message layer. For this reason, it is important that signed data is provided with time of issue in order to prevent replay attacks where an adversary attempts to serve outdated trust information.


### Future migration

The infrastructure specified in this specification is expected to be a temporary solution and its design is focused on allowing Participant Countries to use the system with a minimum amount of efforts and development using currently available standard tools.

One important design goal is therefore to adopt a design that allows this model to be migrated into almost any future model for trust exchange. By gradually moving from self-signed certificates towards a more layered model where each DCA is signed by a national CSCA, it is possible to merge this infrastructure with existing models for trust exchange such as a fully integrated ICAO model, integration with EU trusted list or other relevant models.
