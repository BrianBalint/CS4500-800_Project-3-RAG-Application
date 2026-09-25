# The Datacenter Spec Assistant

**CS4500-800 CIT Seminar, Fall 2026, Dr. Nan Wang, Brian Balint**

**Project 3: RAG Application**

An open-book assistant for Open Compute Project hardware specifications. Ask it
about rack fasteners, busbar voltage or corrosion testing and it retrieves the
relevant passages from the specification PDFs, puts them in front of a language
model, and answers from those passages with a citation to the document and page.

The model is not asked to remember anything. It is asked to read.

## What is here

- Brian Balint CS4500-800_Project 3.ipynb (the application, with outputs from a full run)
- ocp_specs/ (the knowledge base, three OCP specifications totaling 111 pages)

## The corpus

| Document | Pages | Covers |
|---|---|---|
| Open Rack Base Specification Version 3, rev 1.1 | 28 | Mechanical: geometry, fasteners, corrosion, rolling tests |
| OCP Open Rack V3 HPR V2 Power Monitoring Module (PMM), rev 1.0.0 | 33 | Electrical and firmware: Modbus, CAN bus, edge connectors, MCU |
| Open Rack V3 HPR V2 12kW PSU Module, v1.0.0 | 50 | Power: efficiency curves, power factor, current sharing, thermal |

Published by the Open Compute Project under CC BY 4.0 and the OCP Hardware
License, both of which permit redistribution.

## Running it

Everything runs locally against Ollama. Nothing leaves the machine.

    ollama pull nomic-embed-text
    ollama pull qwen2.5:7b-instruct
    pip install pypdf numpy requests

The notebook expects the PDFs in ~/ocp_specs. Change DOCS_DIR in the setup cell
to point somewhere else.

## How it works

Text is extracted page by page, because the page number is what makes a citation
useful. Running headers, footers and invisible bidirectional marks are stripped
at ingestion, since that boilerplate otherwise lands in every chunk and dilutes
the embeddings. Pages are split into overlapping 1,200 character windows cut at
sentence boundaries, so a requirement that straddles a boundary is not lost from
both pieces.

Each passage is embedded with nomic-embed-text and normalized, so ranking a
question against the corpus is a single matrix multiplication. The top five
passages go into the prompt with their document and page labels, and the model
is instructed to answer only from them, to cite every fact, and to say "The
provided specifications do not cover this." when the answer is not present.

Generation runs at temperature 0, so results are reproducible.

## Verifying it uses the documents

Every test question is asked twice, once with the retrieved passages and once
with none. The divergence is the point.

Asked about the ground path corrosion requirement, the model without documents
answered "at least 1,000 hours" of salt spray. The specification says 48 hours,
to rust grade 6 per ASTM D610-01. The out of book answer arrived with a real
ASTM standard number attached and nothing signalling uncertainty.

The final test question asks about Tier IV hot aisle containment height, which
appears in none of the three specifications. Without documents the model
answered it in full. With documents it declined.
