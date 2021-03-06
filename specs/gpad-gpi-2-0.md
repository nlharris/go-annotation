
# Proposed specifications for Gene Ontology Consortium GPAD and GPI tabular formats version 2.0

Table of contents
  * [Abstract](#abstract)
  * [Status](#status)
  * [Summary of changes relative to 1.1](#summary-of-changes-relative-to-11)
- [Outline](#outline)
  * [Preliminary Definitions](#preliminary-definitions)
    + [BNF Notation](#bnf-notation)
    + [Basic Characters](#basic-characters)
    + [Spacing Characters](#spacing-characters)
    + [Identifiers](#identifiers)
    + [GO database registry](#go-database-registry)
    + [Property Symbols](#property-symbols)
    + [Dates](#dates)
- [GPAD Syntax](#gpad-syntax)
    + [GPAD Document Structure](#gpad-document-structure)
    + [GPAD Headers](#gpad-headers)
    + [Annotations](#annotations)
  * [GPAD columns](#gpad-columns)
- [GPI 2.0 Specs](#gpi-20-specs)
  * [GPI Headers](#gpi-headers)
  * [GP Entities](#gp-entities)
  * [GPI Columns](#gpi-columns)
  
## Abstract
This document specifies the syntax of Gene Product Annotation Data (GPAD) and Gene Product Information (GPI) formats. GPAD describes the relationships between biological entities (such as gene products) and biological descriptors (such as GO terms). GPI describes the biological entities.

## Status
This is a working draft, intended for comment by the community.
Comments should be added to: https://github.com/geneontology/go-annotation/issues/2864 

## Summary of changes relative to 1.1
  ### - GPAD and GPI: columns 1 and 2 are now combined in a single column containing an id in CURIE syntax, e.g. UniProtKB:P56704.
  ### - GPAD: negation is captured in a separate column, column 2, using the text string 'NOT'.
  ### - GPAD: gene product-to-term relations captured in column 3 use a Relations Ontology (RO) identifier instead of a text string
  ### - GPAD: the With/From column, column 7, may contain identifiers separated by commas as well as pipes.
  ### - GPAD and GPI: NCBI taxon ids are prefixed with 'NCBITaxon:' to indicate the source of the id, e.g. NCBITaxon:6239
  ### - GPAD: Annotation Extensions in column 11 will use a Relation_ID, rather than a Relation_Symbol, in the Relational_Expression, e.g. RO:0002233(UniProtKB:Q00362)
  ### - GPAD and GPI: dates follow the ISO-8601 format, e.g. YYYY-MM-DD; time may be included as YYYY-MM-DDTHH:MM
  ### - GPI: the entity type in column 5 is captured using an ID from the Sequence Ontology, Protein Ontology, or Gene Ontology.  
  ### - GPI: the parent object id in column 7 refers to the gene-centric parent, e.g. the UniProtKB Gene-Centric Reference Proteome accession or a Model Organism Database gene identifier
  ### - Extensions in file names are: \*.gpad and \*.gpi 
  
# Outline

We first start with some preliminary definitions, including a
description of the notation used in this specification.

The body of the document is split in two - the first part defines GPAD
syntax, the second defines GPI syntax.

## Preliminary Definitions

### BNF Notation

GPAD and GPI document structures are defined using a standard BNF notation, which is summarized below.

 * terminal symbols are single quoted
 * non-terminal symbols are unquoted
 * zero or more symbols are indicated by following the symbol with a star; e.g. `Annotation*`
 * zero or one symbols are written using square brackets; e.g. `[Extension_Conj]`
 * alternative symbols are written using vertical bars
 * groupings are written using parentheses
 * complementation is written using minus symbol

GPI and GPAD documents consist of sequences of Unicode characters and are encoded in ASCII.

### Basic Characters

    Alpha_Char ::= a |b |c |d |e |f |g |h |i |j |k |l |m |n |o |p |q |r |s |t |u |v |w |x |y |z
       | A |B |C |D |E |F |G |H |I |J |K |L |M |N |O |P |Q |R |S |T |U |V |W |X |Y |Z
    Digit ::= 0 |1 |2 |3 |4 |5 |6 |7 |8 |9
    Alphanumeric_Char ::= Alpha_Char | Digit
    Alphanumeric ::= Alphanumeric_Char+

### Spacing Characters

There is a single space character allowed

    Space ::= ' '

### Identifiers

An identifier consists of a prefix and a local identifier separated by a colon symbol

    ID ::= Prefix ':' Local_ID

A Prefix must not contain special characters such as ':'s

    Prefix ::= Alphanumeric | '_' | '-'

A local identifier can consist of any non-whitespace character

    Local_ID ::= (Any_ASCII_Character - ws)

An OBO ID is a type of identifier

    OBO_ID ::= ID

OBO identifiers (which include GO identifiers) SHOULD follow the [OBO identifier policy](http://www.obofoundry.org/id-policy.shtml)

References are also types of identifier

    Reference ::= ID

### GO database registry (db-xrefs.yaml)

The [GO database
registry](https://github.com/geneontology/go-site/blob/master/metadata/db-xrefs.yaml) contains a
list of valid prefixes that can be used in GPAD or GPI files. Every
identifier used in a GPAD or GPI file MUST have an entry in the
registry.

The combination of prefix plus Local_ID (see previous section)
describes how an identifier should be mapped to a URI.

### Property Symbols

Open-ended property-value pairs are allowed at different points
throughout a document. The property symbol or "tag" is a shorthand way
of specifying a URI that denotes the actual property used.

    Property_Symbol ::= ID | Alphanumeric

### Dates

Dates are written into what is equivalent to the date portion of ISO-8601, with the additional constraint that the YMD portion must conform to the regexp /([12]\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01]))/, i.e. “hyphenated” YMD, e.g. 2020-03-10.

In most cases, only the year, month, and date will be lifted and used within the data ingest; additional information, e.g. time, is optional, but may be useful for tracking annotation history as part of annotation imports into Noctua.

# GPAD Syntax

### GPAD Document Structure

A GPAD document consists of a header followed by zero or more
annotations

    GPAD_Doc ::= GPAD_Header Annotation*


### GPAD Headers

A header consists of an obligatory format version declaration followed
by zero or more metadata lines:

    GPAD_Header ::= '!gpa-version: 2.0' nl
                    GPAD_Header_Line*

Each metadata line starts with an exclamation mark '!'. One mark
indicates a structured tag-value pair, two marks indicates free text.

    GPAD_Header_Line ::=
       '!' Property_Symbol ':' Space* Value nl |
       '!!' (Char - nl)* nl

The list of allowed property symbols is open-ended, however several
properties are required:

 * generated-by: database listed in dbxrefs.yaml
 * date-generated: YYYY-MM-DD or YYYY-MM-DDTHH:MM

Groups may decide to include additional information. Examples include:

 * URL: e.g. http://www.yeastgenome.org/
 * Project-release: e.g. WS275
 * Funding: e.g. NHGRI
 * Columns: file format written out
 * go-version: PURL
 * ro-version: PURL
 * gorel-version: PURL
 * eco-version: PURL

### Annotations

In this specification, an annotation is an association between a
biological entity (such as a gene or gene product) and an ontology
class (term). The association describes some aspect of that entity,
and includes with metadata about the association, such as evidence and
provenance.

Each annotation is on a separate line of tab separated values:

    Annotation ::= Col_1 tab Col_2 tab ... Col_12 nl

## GPAD columns

 Each of these columns has its own syntax, as specified below:
 
 Column 	| Content 	| Ontology  | Cardinality | Example ID | Comment
--------|----------|----------- | -------------- | ----------| ---------- |
 1 | DB_Object_ID ::= ID | | 1 | UniProtKB:P11678 |
 2 | Negation ::= 'NOT' | | 0 or 1 | NOT |
 3 | Relation ::= OBO_ID | Relations Ontology (subset, see table below) | 1 | RO:0002263 |
 4 | Ontology_Class_ID ::= OBO_ID | Gene Ontology | 1 | GO:0050803 |
 5 | Reference ::= ID ('\|' ID)* | | 1 or greater | PMID:30695063 | Different IDs, e.g. PMID and MOD paper id, must correspond to the same publication or reference
 6 | Evidence_type ::= OBO_ID | Evidence and Conclusion Ontology | 1 | ECO:0000315 |  Mapping file in progress:  https://github.com/evidenceontology/evidenceontology/issues/249
 7 | With_or_From ::= [ID] ('\|' \| ‘,’ ID)* | | 0 or greater | WB:WBVar00000510 | Pipe-separated entries represent independent evidence; comma-separated entries represent grouped evidence, e.g. two of three genes in a triply mutant organism
 8 | Interacting_taxon_ID ::= ['NCBITaxon:'Taxon_ID] ('\|' \| ‘,’ 'NCBITaxon:'[Taxon_ID])* | | 0 or greater | NCBITaxon:5476 |
 9 | Date ::= YYYY-MM-DD | | 1 | 2019-01-30 |  
10 | Assigned_by ::= Prefix | | 1 or greater | MGI |
11 | Annotation_Extensions ::= [Extension_Conj] ('\|' Extension_Conj)* | | 0 or greater | BFO:0000066(GO:0005829) |   
12 | Annotation_Properties ::= [Property_Value_Pair] ('\|' Property_Value_Pair)* | | 0 or greater | contributor-id=https://orcid.org/0000-0002-1478-7671 |

    Taxon_ID ::= Digit+
    
    The Taxon_ID should be a taxon identifier from the NCBI Taxonomy database.
    
    Extension_Conj ::= [Relational_Expression] (',' Relational_Expression)*

    Relational_Expression ::= Relation_ID '(' ID ')'
    
    Relation_ID ::= ID
    
    The Relation_ID may be from the Relations Ontology or from the set of GO relations, go_rel.

    Property_Value_Pair ::= Property_Symbol '=' Property_Value

    Property_Value  ::= (AnyChar - ('=' | '|' | nl))
    ASCII, except for non-printing characters, tabs, new lines, control characters, quotation marks, or pipe
    
### Allowed Gene Product to GO Term Relations

#### Default usage is indicated for MF and CC.  Groups may choose which relation to use for BP annotations according to their curation practice.  'acts upstream of or within' is the parent Relations Ontology term for the BP relations listed below.  A full view of the BP relation hierarchy can be found at http://www.ontobee.org/ or https://www.ebi.ac.uk/ols/index. Note: the RO term labels and IDs listed below are current as of 2020-06-09.  However, to ensure accurate use of RO, groups should always derive mappings between RO term labels and IDs from the RO source file available here: https://github.com/oborel/obo-relations


GO Aspect 	| Relations Ontology Label  | Relations Ontology ID | Usage Guidelines
-----------|---------------------------|----------------------| ------------------ |
Molecular Function | enables | RO:0002327 | Default for MF
Molecular Function | contributes to | RO:0002326 |
Biological Process | involved in | RO:0002331 |
Biological Process | acts upstream of | RO:0002263 |
Biological Process | acts upstream of positive effect | RO:0004034 |
Biological Process | acts upstream of negative effect | RO:0004035 |
Biological Process | acts upstream of or within | RO:0002264 |
Biological Process | acts upstream of or within positive effect | RO:0004032 |
Biological Process | acts upstream of or within negative effect | RO:0004033 |
Cellular Component | part of	| BFO:0000050 | Default for protein-containing complex (GO:0032991) and child terms
Cellular Component | located in | RO:0001025 | Default for non-protein-containing complex CC terms
Cellular Component | is active in | RO:0002432 | Used to indicate where a gene product enables its MF
Cellular Component | colocalizes with | RO:0002325 |

### GPAD Annotation Properties (Proposed)

Annotation_Property_Symbol | Property_Value | Cardinality (if used) | Example | Semantics 
---------------------------|----------------|------------ | ------- | --------- |
 id | unique database identifier | 1 | id=WBOA:3219 | Unique identifier for an annotation in a contributing database. |
 model-state | GO-CAM model state | 1 | model-state=production |
 noctua-model-id | unique GO-CAM model id | 1 | noctua-model-id=gomodel:5a7e68a100001078 |
 contributor-id | ORCID | 1 | contributor-id=https://orcid.org/0000-0002-1706-4196 | Used by GOC to indicate ORCID of curator or user who entered or changed an annotation |
 reviewer-id | ORCID | 1 | reviewer-id=https://orcid.org/0000-0001-7476-6306 | Used by GOC to indicate ORCID of curator or user who last reviewed an annotation |
 creation-date | YYYY-MM-DD | 1 | creation-date=2019-02-05 | The date on which the annotation was created. |
 modification-date | YYYY-MM-DD | 1 | modification-date=2019-02-06 | The date(s) on which an annotation was modified. |
 reviewed-date | YYYY-MM-DD | 1 | reviewed-date=2019-02-06 | The date(s) on which the annotation was reviewed. |
 comment | text | 1 | comment=Confirmed species by checking PMID:nnnnnnnn. | Free-text field that allows curators or users to enter notes about a specific annotation. |

    
# GPI 2.0 Specs 

### GPI Document Structure

A GPI document consists of a header followed by zero or more
annotations

    GPI_Doc ::= GPI_Header Annotation*

## GPI Headers

A header consists of an obligatory format version declaration followed
by an obligator database declaration then zero or more lines starting
with an exclamation point:

    GPI_Header ::= '!gpi-version: 2.0' nl
                   '!namespace: ' Prefix nl
                   Header_Line*

Each metadata line starts with an exclamation mark '!'. One mark
indicates a structured tag-value pair, two marks indicates free text.

    GPI_Header_Line ::=
       '!' Property_Symbol ':' Space* Value nl |
       '!!' (Char - nl)* nl
      
The list of allowed property symbols is open-ended, however several
properties are required:

 * generated-by: database listed in dbxrefs.yaml
 * date-generated: YYYY-MM-DD or YYYY-MM-DDTHH:MM
 
Groups may decide to include additional information. Examples include:

 * URL: e.g. http://www.yeastgenome.org/
 * Project-release: e.g. WS275
 * Funding: e.g. NHGRI
 * Columns: file format written out
 * DB-Object-Type-information, e.g. DB_Object_Type = "gene", DB = "MGI", DB_Object_ID = "MGI:xxxx"
       
## GP Entities

A GP entity is any biological entity that can be annotated using GPAD


Each entity is written on a separate line of tab separated values:

    Entity ::= Col_1 tab Col_2 tab ... Col_11 nl
    
    
## GPI Columns

 Column 	| Content 	| Ontology  | Cardinality | Example ID | Comments
--------|----------|-----------|-----------|-----------|-----------|
1 | DB_Object_ID ::= ID      | | 1 | UniProtKB:Q4VCS5 | |
2 | DB_Object_Symbol ::= xxxx      | | 1 | AMOT | |
3 | DB_Object_Name ::= [Label] ('\|' Label)*  | | 0 or greater | Angiomotin | |
4 | DB_Object_Synonyms ::= [Label] ('\|' Label)*  | | 0 or greater | AMOT\|KIAA1071 | | 
5 | DB_Object_Type ::= OBO_ID ('\|' OBO_ID)* | Sequence Ontology OR Protein Ontology OR Gene Ontology | 1 or greater | PR:000000001 | | If a gene encodes for both protein and ncRNA, more than one type can be applied.
6 | DB_Object_Taxon ::= NCBITaxon:[Taxon_ID] || 1 |  NCBITaxon:9606 | | 
7 | Encoded_By ::= [ID] ('\|' ID)* | | 0 or greater | HGNC:17810  | For proteins and transcripts, this refers to the gene id that encodes those entities. | 
8 | Parent_Protein ::= [ID] ('\|' ID)* | | 0 or greater |  | When column 1 refers to a protein isoform or modified protein, this column refers to the gene-centric reference protein accession of the column 1 entry. | 
9 | Protein_Containing_Complex_Members ::= [ID] ('\|' ID)* | | 0 or greater | UniProtKB:Q15021\|UniProtKB:Q15003 | 
10 | DB_Xrefs ::= [ID] ('\|' ID)* | | 0 or greater | HGNC:17810 | See below for required DB xref values |
11 | Gene_Product_Properties ::= [Property_Value_Pair] ('\|' Property_Value_Pair)* |  | 0 or greater | db-subset=Swiss-Prot  | |

    Property_Value_Pair ::= Property_Symbol '=' Property_Value

    Property_Value  ::= (AnyChar - ('=' | '|' | nl))
    ASCII, except for non-printing characters, tabs, new lines, control characters, quotation marks, or pipe
    
### GPI Entity Types 
#### Entity types may be one of the following, or a more granular child term.

Entity Type | Ontology Label | Ontology ID 
---------------------------|----------------|------------ | 
protein-coding gene | protein_coding_gene | SO:0001217 
ncRNA-coding gene | ncRNA_gene  | SO:0001263 
mRNA | mRNA | SO:0000234
ncRNA | ncRNA | SO:0000655 
protein | protein | PR:000000001
protein-containing complex | protein-containing complex | GO:0032991
marker or uncloned locus | genetic_marker | SO:0001645

Other possible entity types from MGI (additional examples coming):

 *gene segment: SO:3000000
 
 *pseudogene: SO:0000336
 
 *Example: http://www.informatics.jax.org/marker/MGI:3029152
 
 *gene: SO:0000704
 
 *biological region: SO:0001411


### Required and Optional DB xrefs
#### Required:

 MODs: Must associate gene ids, for protein-coding genes, with UniProtKB gene-centric reference protein accessions
 
 UniProtKB: Must associate gene-centric reference protein accessions with MOD gene ids

#### Optional DB xref suggestions (where applicable):

 *RNAcentral 
 
 *Ensembl gene
 
 *NCBI RefSeq gene
 
 *HGNC
 
 *ComplexPortal
 
 *PRO

### GPI Gene Product Properties (Proposed)

Annotation_Property_Symbol | Property_Value | Cardinality (if used) | Example | Semantics 
---------------------------|----------------|------------ | ------- | --------- |
db-subset | TrEMBL or Swiss-Prot | 1 | db-subset=TrEMBL | The status of a UniProtKB accession with respect to curator review.
uniprot-proteome | identifier  | 1 | uniprot-proteome=UP000001940 | A unique UniProtKB identifier for the set of proteins that constitute an organism's proteome.
go-annotation-complete | YYYY-MM-DD | 1| 2019-02-05 | Indicates the date on which a curator determined that the set of GO annotations for a given entity is complete with respect to GO annotation.  Complete means that all information about a gene has been captured as a GO term, but not necessarily that all possible supporting evidence is annotated.
go-annotation-summary | text | 1 | go-annotation-summary=Sterol binding protein with a role in intracellular sterol transport; localizes to mitochondria and the cortical ER | A textual gene or gene product description.
