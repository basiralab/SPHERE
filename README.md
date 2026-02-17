# SPHERE: Scientific Paper Hierarchical Entity and Relation Extraction

SPHERE is a large-scale, multi-domain benchmark dataset for hierarchical relation extraction from scientific text. It is designed to train and evaluate models that construct structured knowledge graphs from scientific literature, capturing not just flat entity-relation triples but deep hierarchical dependencies between concepts.

## Motivation

Existing scientific information extraction benchmarks such as SciERC and SciER are limited to a few hundred abstracts in a single domain, with flat entity ontologies and locally inferred hierarchies. SPHERE addresses these limitations by providing:

- **Multi-domain coverage** across four scientific fields: Computer Science, Physics, Biology, and Material Science
- **Global hierarchical consistency** via a pre-defined Knowledge Graph scaffold, ensuring that the hierarchical position of every concept is consistent across the entire corpus
- **Deep taxonomic structure** with nested, multi-level concept hierarchies (e.g., *Adam Optimizer* &rarr; *Stochastic Optimization* &rarr; *Optimization Method*)
- **Three canonical relation types**: `is-a-subconcept-of`, `related-to`, and `dependent-on`

## Dataset Structure

```
SPHERE/
├── computer_science/
│   ├── computer_science.jsonl                 # Annotated sentences
│   └── final_cs_knowledge_graph.json          # Knowledge graph scaffold
├── physics/
│   ├── annotated_physics_sentences.jsonl
│   └── final_physics_knowledge_graph.json
├── biology/
│   ├── annotated_biology_sentences.jsonl
│   └── final_biology_knowledge_graph.json
└── material_science/
    ├── annotated_materials_sentences.jsonl
    └── final_materials_knowledge_graph_4_levels.json
```

### Sentence Annotations (`.jsonl`)

Each line is a JSON object representing one annotated sentence:

```json
{
  "text": "Backpropagation computes gradients using the chain rule for training deep neural networks.",
  "entities": [
    {"id": 42, "name": "Backpropagation", "span": [0, 15], "type": "Algorithm"},
    {"id": 97, "name": "Chain Rule", "span": [46, 56], "type": "Mathematical Concept"},
    {"id": 113, "name": "Deep Neural Networks", "span": [70, 90], "type": "Model"}
  ],
  "relations": [
    {"source": 42, "target": 97, "type": "dependent-on"},
    {"source": 42, "target": 113, "type": "related-to"}
  ],
  "sentence_id": 1
}
```

**Fields:**
- `text` &mdash; The sentence text
- `entities` &mdash; List of entity mentions, each with:
  - `id` &mdash; Permanent concept ID from the knowledge graph
  - `name` &mdash; Canonical entity name (resolved to the KG concept, may include disambiguation qualifiers)
  - `span` &mdash; Character offsets `[start, end)` in the sentence text (0-based, exclusive end)
  - `type` &mdash; Entity type from the KG taxonomy
- `relations` &mdash; List of relations between entities, each with:
  - `source` / `target` &mdash; Entity IDs
  - `type` &mdash; One of `is-a-subconcept-of`, `related-to`, or `dependent-on`

### Knowledge Graphs (`.json`)

Each domain has a knowledge graph file containing the full concept taxonomy:

```json
[
  {
    "name": "Artificial Intelligence",
    "type": "Sub-field",
    "relations": [
      {"target": "Machine Learning", "type": "is-a-subconcept-of"},
      {"target": "Computer Science", "type": "is-a-subconcept-of"}
    ]
  }
]
```

## Generation Methodology

SPHERE is generated via a three-phase pipeline:

1. **Programmatic KG Scaffolding** &mdash; A ground-truth knowledge graph is constructed by prompting an LLM to recursively expand a high-level field into a deep, interconnected taxonomy of sub-fields, methods, and concepts.

2. **High-Throughput Sentence Generation** &mdash; Contextually related sets of concepts are sampled from the KG, and an LLM generates academic-style paragraphs describing their relationships.

3. **LLM Self-Annotation** &mdash; The generated sentences are passed back to the LLM for Named Entity Recognition and Relation Extraction, linking identified concepts back to their permanent IDs in the ground-truth KG.

## Relation Types

| Type | Description | Example |
|------|-------------|---------|
| `is-a-subconcept-of` | Hierarchical/taxonomic link | *Deep Learning* is-a-subconcept-of *Machine Learning* |
| `related-to` | General semantic association | *Dropout* related-to *Regularization* |
| `dependent-on` | Foundational dependency | *Backpropagation* dependent-on *Chain Rule* |

## Usage

SPHERE is designed as a benchmark for:

- **Named Entity Recognition (NER)** in scientific text
- **Relation Extraction (RE)** with hierarchical relation types
- **Zero-shot domain transfer** &mdash; train on one domain, evaluate on another
- **Scientific knowledge graph construction** at scale

## Citation

```bibtex
@inproceedings{
joshi2026hgnet,
title={{HGN}et: Scalable Foundation Model for Automated Knowledge Graph Generation from Scientific Literature},
author={Devvrat Joshi and Islem Rekik},
booktitle={The Fourteenth International Conference on Learning Representations},
year={2026},
url={https://openreview.net/forum?id=NWd53rltx8}
}
```

## License

This dataset is released for research purposes. Please cite the paper above if you use SPHERE in your work.
