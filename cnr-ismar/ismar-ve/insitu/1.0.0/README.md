# ISMAR Dataset Metadata Application Profile — draft

Questo repository contiene una prima bozza minimale di un **metadata application profile** espresso come file RDF/Turtle con vincoli **SHACL** incorporati direttamente nella definizione dei singoli campi.

L'obiettivo non è costruire fin da subito un'ontologia completa, ma avere un file leggibile e progressivamente estendibile che possa essere referenziato dai dataset tramite metadati come:

```text
metadata_profile = "https://example.org/profiles/ismar-dataset-metadata/1.0.0/profile.ttl"
metadata_profile_version = "1.0.0-draft"
Conventions = "CF-1.13, ACDD-1.3, ISMAR-MAP-1.0.0-draft"
```

> **Nota:** gli URL `https://example.org/...` sono segnaposto e vanno sostituiti con l'URI persistente definitivo del profilo.

---

## File inclusi

- `profile.ttl`  
  File RDF/Turtle canonico del profilo. Contiene:
  - metadati generali del profilo;
  - riferimenti agli standard/profili di base;
  - shape globale del dataset;
  - shape per attributi globali;
  - shape per attributi di variabile;
  - definizioni testuali e vincoli SHACL direttamente nei singoli campi.

---

## Principio di modellazione

La scelta adottata è volutamente semplice:

- ogni campo è rappresentato come una risorsa `sh:PropertyShape`;
- la stessa risorsa contiene:
  - nome del campo;
  - descrizione;
  - indicazione se il campo è globale o di variabile;
  - obbligatorietà, quando esprimibile;
  - datatype, cardinalità, pattern o range, quando esprimibili in SHACL;
  - note testuali per ciò che per ora non viene validato automaticamente.

Esempio:

```turtle
ismap:title
    a sh:PropertyShape ;
    sh:path ismap:title ;
    sh:name "title" ;
    skos:definition "A short human-readable title describing the dataset."@en ;
    skos:scopeNote "Global attribute. Mandatory. Must not be empty."@en ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
    sh:datatype xsd:string ;
    sh:minLength 1 .
```

Questa impostazione privilegia la leggibilità rispetto a una separazione ontologica più articolata tra proprietà, campi, shape, vocabolari e regole applicative.

---

## Vocabolari usati

Il file usa solo vocabolari esistenti:

- **SHACL** (`sh:`) per descrivere vincoli validabili: cardinalità, datatype, pattern, range numerici, messaggi;
- **SKOS** (`skos:`) per definizioni, etichette e note d'uso;
- **Dublin Core Terms** (`dct:`) per titolo, descrizione, creator, versione, licenza e riferimenti;
- **W3C Profiles Vocabulary** (`prof:`) per dichiarare che la risorsa è un profilo;
- **RDFS** (`rdfs:`) per collegamenti descrittivi generali;
- **XSD** (`xsd:`) per i datatype.

Non è stata introdotta una ontologia locale per descrivere concetti come “global attribute”, “variable attribute”, “mandatory” o “recommended”. Queste informazioni sono per ora espresse in modo leggibile tramite `skos:scopeNote` e, dove possibile, tramite vincoli SHACL.

---

## Obbligatorietà

La bozza usa questa convenzione:

- campi obbligatori: `sh:minCount 1`;
- campi opzionali: nessun `sh:minCount`;
- campi raccomandati: nessun `sh:minCount`, ma `sh:severity sh:Warning` quando utile;
- campi condizionali: descrizione testuale in `skos:scopeNote`, con eventuale validazione rimandata a strumenti successivi.

Esempio di campo obbligatorio:

```turtle
sh:minCount 1 ;
sh:maxCount 1 .
```

Esempio di campo raccomandato:

```turtle
sh:maxCount 1 ;
sh:severity sh:Warning .
```

---

## Campi globali iniziali

La bozza include, tra gli altri:

- `title`
- `summary`
- `institution`
- `creator_name`
- `creator_email`
- `metadata_profile`
- `metadata_profile_version`
- `Conventions`
- `keywords`
- `license`
- `sourceUrl`
- `infoUrl`
- `time_coverage_start`
- `time_coverage_end`
- `geospatial_lat_min`
- `geospatial_lat_max`
- `geospatial_lon_min`
- `geospatial_lon_max`

---

## Campi di variabile iniziali

La bozza include:

- `long_name`
- `standard_name`
- `units`
- `ioos_category`
- `_FillValue`
- `missing_value`

Per `_FillValue` e `missing_value`, la bozza non tenta di validare completamente il rapporto tra valore del metadato e datatype della variabile. Questa parte è lasciata intenzionalmente a futuri validatori applicativi, perché non è banale esprimerla in modo semplice con SHACL senza introdurre un modello RDF più articolato del dataset.

---

## Come collegare un dataset al profilo

Per un dataset netCDF/ERDDAP/THREDDS si suggerisce di usare almeno:

```text
metadata_profile = "https://example.org/profiles/ismar-dataset-metadata/1.0.0/profile.ttl"
metadata_profile_version = "1.0.0-draft"
Conventions = "CF-1.13, ACDD-1.3, ISMAR-MAP-1.0.0-draft"
```

Quando sarà disponibile un URI persistente definitivo, sostituire `https://example.org/...` con l'URI reale.

---

## Cosa resta fuori, per ora

Questa bozza non cerca ancora di risolvere:

- validazione completa di file netCDF reali;
- conversione automatica netCDF → RDF;
- vincoli condizionali complessi;
- controllo effettivo che `_FillValue` abbia lo stesso datatype della variabile;
- controllo contro vocabolari esterni, ad esempio CF Standard Names o liste controllate ERDDAP;
- generazione automatica di documentazione HTML.

Questi aspetti potranno essere affrontati in una fase successiva.

---

## Prossimi passi suggeriti

1. Sostituire l'URI base `https://example.org/profiles/ismar-dataset-metadata/1.0.0#` con l'URI definitivo.
2. Rivedere l'elenco dei campi globali e di variabile.
3. Decidere quali campi siano realmente obbligatori, raccomandati, opzionali o condizionali.
4. Uniformare le regole per stringa vuota, omissione, `N/A`, `None` e valori mancanti.
5. Aggiungere progressivamente i mapping verso CF, ACDD, DataCite, DCAT o altri riferimenti utili.
6. In una fase successiva, valutare un piccolo validatore che legga i metadati reali e applichi le parti SHACL effettivamente traducibili.

---

## Licenza

La bozza indica come licenza predefinita [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/). Modificarla se il repository o l'ente richiedono una licenza diversa.
