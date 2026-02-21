# EPO OPS

European Patent Office Open Patent Services API.

**Required**: `EPO_OPS_API_KEY` and `EPO_OPS_API_SECRET` environment variables.

## Client

```python
from ip_tools.epo_ops import EpoOpsClient, client_from_env, get_client

# From environment variables
async with client_from_env() as client:
    ...

# Or explicit credentials
async with EpoOpsClient(api_key="...", api_secret="...") as client:
    ...
```

## Methods

### search_published(query, range_begin, range_end) -> SearchResponse

Search published documents using CQL.

```python
async with client_from_env() as client:
    results = await client.search_published(
        query='ta="machine learning" and pd within "20200101,20231231"',
        range_begin=1,
        range_end=25
    )

    for doc in results.results:
        doc.doc_number
        doc.country
        doc.kind
        doc.family_id
    results.total_results  # Total number of results
```

### search_families(query, range_begin, range_end) -> FamilySearchResponse

Search grouped by patent family.

```python
results = await client.search_families(
    query='applicant="Google"',
    range_begin=1,
    range_end=25
)
```

### fetch_biblio(number) -> BiblioResponse

Get bibliographic data. Returns a response with a list of `BiblioRecord` documents.

```python
biblio = await client.fetch_biblio(number="EP1234567A1")
for doc in biblio.documents:
    doc.title
    doc.abstract
    doc.applicants
    doc.inventors
    doc.ipc_classes
    doc.cpc_classes
    doc.priority_claims
    doc.publication_reference
    doc.application_reference
    doc.family_id
```

### fetch_fulltext(number, section) -> FullTextResponse

Get claims or description.

```python
# Get claims
claims = await client.fetch_fulltext(number="EP1234567A1", section="claims")
for claim in claims.claims:
    claim.number
    claim.text
    claim.depends_on

# Get description
desc = await client.fetch_fulltext(number="EP1234567A1", section="description")
desc.text
```

### fetch_family(number) -> FamilyResponse

Get patent family members.

```python
family = await client.fetch_family(number="EP1234567A1")
for member in family.members:
    member.family_id
    member.publication_number
    member.application_number
    member.publication_references  # List of publication references
    member.application_references
    member.priority_claims
```

### fetch_legal_events(number) -> LegalEventsResponse

Get legal status history.

```python
events = await client.fetch_legal_events(number="EP1234567A1")
for event in events.events:
    event.event_code
    event.free_text
    event.event_date
    event.event_country
```

### download_pdf(number) -> PdfDownloadResponse

Download patent PDF.

```python
pdf = await client.download_pdf(number="EP1234567A1")
pdf.pdf_base64  # Base64-encoded PDF
```

### retrieve_cpc(cpc_code) -> CpcRetrievalResponse

Get CPC classification details.

```python
cpc = await client.retrieve_cpc("G06N3/08")
cpc.title
cpc.definition
```

### search_cpc(query) -> CpcSearchResponse

Search CPC classifications.

```python
results = await client.search_cpc("neural network")
```

## EP Register Methods

The EP Register provides prosecution-specific data for European applications.

### search_register(query, range) -> RegisterSearchResponse

Search EP Register for European applications.

```python
results = await client.search_register(
    query='pa="Siemens"',  # Applicant search
    range_begin=1,
    range_end=25
)

for app in results.results:
    app.application_number
    app.publication_number
    app.title
    app.status
```

### fetch_register_biblio(number) -> RegisterBiblioResponse

Get detailed EP Register data for an application.

```python
biblio = await client.fetch_register_biblio(
    number="EP20200001",
    doc_type="application",
    fmt="epodoc"
)

biblio.application_number
biblio.publication_number
biblio.application_date
biblio.grant_date
biblio.title
biblio.applicants
biblio.inventors
biblio.representatives
biblio.status
biblio.status_description

# Designated states
for state in biblio.designated_states:
    state.country_code
    state.status
    state.effective_date

# Opposition data (if applicable)
if biblio.opposition:
    biblio.opposition.opposition_date
    biblio.opposition.status
    for party in biblio.opposition.parties:
        party.name
        party.role  # "opponent" or "patent_owner"
```

### fetch_register_procedural_steps(number) -> RegisterProceduralStepsResponse

Get prosecution history (procedural steps).

```python
steps = await client.fetch_register_procedural_steps("EP20200001")

for step in steps.procedural_steps:
    step.phase
    step.step_code
    step.step_description
    step.step_date
    step.office
```

### fetch_register_events(number) -> RegisterEventsResponse

Get EPO Bulletin events.

```python
events = await client.fetch_register_events("EP20200001")

for event in events.events:
    event.event_code
    event.event_date
    event.event_description
    event.bulletin_number
    event.bulletin_date
```

### fetch_register_upp(number) -> RegisterUppResponse

Get Unitary Patent (UPP) data.

```python
upp = await client.fetch_register_upp("EP20200001")

if upp.unitary_patent:
    upp.unitary_patent.upp_status
    upp.unitary_patent.registration_date
    upp.unitary_patent.participating_states  # List of country codes
```

## CQL Query Syntax

EPO uses Cooperative Query Language. Field abbreviations: `ta` (title/abstract), `pa` (applicant), `in` (inventor), `pd` (publication date), `ipc`/`cpc` (classification).

**IMPORTANT**: For date ranges, always use `pd within "YYYYMMDD,YYYYMMDD"`. Do NOT use `pd>=`/`pd<=` — this causes 413 errors even on small result sets.

```
# Title/abstract search
ta="machine learning"

# Applicant (use pa, not applicant)
pa="Google"
pa all "Inventio AG"   # exact multi-word match

# Inventor
in="Smith"

# Publication date — single date
pd=20240115

# Publication date — range (MUST use 'within', not >= / <=)
pd within "20240101,20240131"

# Classification
cpc="G06N"
ipc="B66B"

# Combined
ta="neural" and pa="IBM" and pd within "20240101,20241231"
```

## Rate Limits

EPO OPS has quota limits:
- **Traffic light**: Check response headers for quota status
- **Weekly quota**: Resets Sunday midnight CET
- **Throttling**: Automatic backoff on 429 responses
