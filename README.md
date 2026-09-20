# ISMAR Metadata Profiles

This repository contains versioned metadata application profiles developed for the management, validation, publication, and interoperability of research data at CNR-ISMAR Venice.

It is the evolution of the earlier [ISMAR-VE Metadata Application Profile](https://github.com/federicobellati/ISMAR-VE_metadata_application_profile). The original repository introduced a single, compact RDF/Turtle profile. This repository extends that work into a structure intended to host multiple profiles, explicit versions, reusable semantic components, and machine-actionable crosswalks to external metadata standards and vocabularies.

## Purpose

The repository provides metadata specifications that are:

- **human-readable**, so that data managers and domain experts can inspect and discuss every field;
- **machine-actionable**, through RDF, SHACL, stable identifiers, and explicit mappings;
- **versioned**, so that datasets can declare the exact profile they implement;
- **extensible**, allowing new profiles and fields to be added without changing previous released versions;
- **interoperable**, through crosswalks to established metadata standards, profiles, and vocabularies;
- **FAIR-oriented**, supporting the consistent description, validation, discovery, and reuse of research data.

The profiles are application profiles rather than attempts to define a new general-purpose ontology. Existing vocabularies are reused whenever possible, while local terms are introduced only where a profile requires an explicit field or rule that is not adequately represented elsewhere.

## Repository organisation

Profiles are organised by subject and version. A version directory represents a fixed release of a profile and should remain stable after publication.

The general pattern is:

```text
<profile-name>/
└── <version>/
    ├── profile.ttl
    └── supporting RDF/Turtle resources, when required
```

At repository level, shared resources may be used for declarations or crosswalks that apply to more than one profile.

The authoritative content is stored in RDF/Turtle files. The Git repository is the development and change-tracking environment, while persistent identifiers should be used by published datasets and external systems whenever available.

## Core modelling approach

Each metadata field is represented as a named `sh:PropertyShape`. The same resource combines the field's semantic documentation with constraints that can be evaluated by a SHACL processor.

A typical field includes:

- a stable IRI;
- `sh:path`, using a property from the profile namespace;
- a human-readable name;
- an English definition and usage notes;
- cardinality constraints;
- datatype, class, pattern, controlled-value, or numerical constraints where applicable;
- validation severity and messages where useful;
- explicit crosswalk statements to external standards.

Example:

```turtle
ismap:title
    a sh:PropertyShape ;
    sh:path ismap:title ;
    sh:name "title"@en ;
    skos:definition "A short human-readable title describing the resource."@en ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
    sh:datatype xsd:string ;
    sh:minLength 1 .
```

This intentionally compact pattern keeps definitions and validation rules close to the field they describe. It favours maintainability and reviewability over a more elaborate separation between ontology terms, form fields, documentation resources, and validation shapes.

## Node shapes and property shapes

A profile normally contains:

- **node shapes**, which identify the type of resource being described and group the applicable fields;
- **property shapes**, which define individual metadata elements;
- **profile metadata**, which identify the profile, version, title, description, licence, and relationships to other specifications;
- **references to external standards and vocabularies**;
- **crosswalk annotations**, when a local field corresponds to an element in another metadata scheme.

Local shape paths use properties in the profile namespace. This keeps the internal data model coherent and prevents the validation model from depending directly on the implementation details of every external standard.

## Requirement levels

SHACL constraints express requirements whenever they can be validated reliably:

- **mandatory fields** use `sh:minCount 1`;
- **single-valued fields** use `sh:maxCount 1`;
- **optional fields** omit `sh:minCount`;
- **recommended fields** may use `sh:severity sh:Warning` without becoming mandatory;
- **conditional requirements** are documented in usage notes and may be implemented with more specific SHACL rules when the condition can be evaluated unambiguously.

Textual documentation remains important. Not every scientifically meaningful recommendation can, or should, be reduced to a syntactic constraint.

## Crosswalks and interoperability

Crosswalks relate profile fields to elements in external metadata standards and vocabularies. They are designed to remain lightweight and machine-actionable without importing the complete data model of every target standard into the application profile.

The mapping predicates identify the target scheme, for example:

```turtle
ismap:title
    ismapmap:iso19115 "CI_Citation.title" ;
    ismapmap:dcatap "dct:title" ;
    ismapmap:datacite "titles/title" ;
    ismapmap:dublincore "dct:title" ;
    ismapmap:schemaorg "schema:name" .
```

The mapping vocabulary should separately declare each mapping predicate and the standard or profile it represents. This distinction is important because the same external term can be reused by several application profiles. For example, `dct:title` may be used both in Dublin Core Terms and in DCAT-AP, while the intended mapping context remains different.

Crosswalk values are identifiers or paths in the target specification. They are not automatically equivalent RDF properties and should not be interpreted as logical equivalence unless that stronger relationship is explicitly asserted.

Mappings can be provided for standards and vocabularies such as:

- ISO 19115;
- DCAT-AP;
- DataCite Metadata Schema;
- Dublin Core Terms;
- INSPIRE metadata;
- Schema.org;
- CF Conventions;
- ACDD;
- SOSA/SSN and related semantic models, where applicable.

A crosswalk supports transformation and metadata exchange, but it does not by itself guarantee lossless conversion. Cardinalities, value structures, controlled vocabularies, and semantic scope may differ between the source and target schemes.

## Main vocabularies

The profiles primarily reuse established Semantic Web vocabularies:

- **SHACL** (`sh:`) for node and property shapes, constraints, severities, and validation messages;
- **SKOS** (`skos:`) for definitions, preferred labels, scope notes, and documentation;
- **Dublin Core Terms** (`dct:`) for descriptive and administrative metadata about profiles;
- **W3C Profiles Vocabulary** (`prof:`) for identifying profiles and their relationships;
- **RDF** and **RDFS** (`rdf:`, `rdfs:`) for the RDF data model and general semantic relationships;
- **XML Schema** (`xsd:`) for datatypes.

Additional vocabularies may be used by individual profiles when they provide the correct domain semantics.

## Versioning and persistence

Profile versions are represented by explicit version directories. Published datasets should reference a specific version rather than an unversioned development resource when reproducibility is required.

A profile release should:

1. have a unique version IRI;
2. declare its version identifier in RDF metadata;
3. preserve previously published versions;
4. record material changes in the Git history and, where appropriate, in release notes;
5. use persistent HTTP identifiers for external references.

Unversioned identifiers may resolve to the current recommended version, but they should not replace versioned references in metadata records that must remain reproducible.

## Referencing a profile from a dataset

A dataset or service can declare the profile and version it follows. The exact encoding depends on the host format. For a NetCDF-oriented workflow, global attributes may follow a pattern such as:

```text
metadata_profile = "<persistent IRI of the versioned profile>"
metadata_profile_version = "<profile version>"
Conventions = "<applicable conventions and profile identifier>"
```

Replace the placeholders with the identifiers declared by the selected profile version. The profile IRI, version value, and `Conventions` entry should be mutually consistent.

## Validation

The Turtle files can be parsed with any standards-compliant RDF library. SHACL validation requires:

1. an RDF data graph containing the metadata to validate;
2. the relevant profile as the SHACL shapes graph;
3. a SHACL processor supporting the constraints used by that profile.

Conceptual command-line example:

```bash
pyshacl -s path/to/profile.ttl -d path/to/metadata.ttl
```

Validation results should be interpreted according to severity:

- `sh:Violation` indicates non-conformance with a required constraint;
- `sh:Warning` identifies a recommended improvement or a non-blocking issue;
- `sh:Info` provides advisory information.

Successful SHACL validation confirms conformance with the encoded rules. It does not replace scientific review, provenance assessment, vocabulary governance, or checks that cannot be expressed in the shapes graph.

## Adding or revising a profile

When contributing a profile or a new version:

1. create or update the appropriate version directory;
2. use stable, dereferenceable IRIs for profile resources and fields;
3. keep `sh:path` values within the profile namespace unless there is a documented reason not to;
4. provide English labels, definitions, and scope notes;
5. encode cardinality and datatype constraints only when they reflect an actual requirement;
6. distinguish mandatory, recommended, optional, and conditional fields clearly;
7. reuse existing vocabularies where the semantic correspondence is sound;
8. update crosswalks without claiming stronger equivalence than the mapping supports;
9. parse the Turtle syntax and run SHACL validation tests before publication;
10. do not modify an already published version in a way that changes its meaning. Create a new version instead.

## Relationship with the previous repository

The earlier [ISMAR-VE Metadata Application Profile](https://github.com/federicobellati/ISMAR-VE_metadata_application_profile) remains useful as the origin of the modelling approach. This repository supersedes it as the development location for the expanded and versioned family of metadata profiles.

The main evolution is from:

- one draft profile in a single file;
- placeholder publication identifiers;
- a primarily self-contained field model;

into:

- multiple independently versioned profiles;
- persistent, referenceable profile resources;
- reusable supporting vocabularies;
- explicit crosswalks to external standards;
- a structure suitable for long-term maintenance and extension.

Consumers should therefore use the profiles and versions published in this repository rather than assuming that the original standalone draft represents the current model.

## Status

The profiles are developed iteratively. A file being present in the repository does not by itself imply that it is a final or endorsed release. Consumers should inspect the profile metadata, version identifier, Git history, and release documentation before using it in production workflows.

Backward compatibility should be evaluated at the profile-version level. Changes to cardinality, datatype, controlled vocabularies, field semantics, or crosswalk targets may require a new version.

## Licence

Unless otherwise stated in a specific file, the repository is made available under the terms of the licence included in [`LICENSE`](LICENSE).

Metadata standards and vocabularies referenced by the profiles remain subject to the terms and governance of their respective publishers.

## Citation and reuse

When reusing a profile, cite the exact versioned profile IRI and the repository. If a profile is used to validate or transform metadata, record the version and validation date in the processing provenance.

Issues and pull requests should describe:

- the affected profile and version;
- the field or shape concerned;
- the proposed semantic or validation change;
- the external specification supporting a crosswalk change;
- any expected compatibility impact.
