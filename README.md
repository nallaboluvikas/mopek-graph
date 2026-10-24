# MOPeK: Multi-Omics Peptide Knowledge Graph 

This package lets you run the complete MOPeK knowledge graph on your own
machine — all 1,232,604 nodes and well over 1,000,000 relationships
across 19 entity types, exactly as described in the accompanying paper.
No cloud account, no size limits, no internet connection required after
the initial download.

## What you need

- **Docker** and **Docker Compose** installed
  (<https://docs.docker.com/get-docker/> if you don't have it yet)
- **At least 2 GB of free RAM** allocated to Docker, and **6 GB of
  free disk space** (the compressed database is ~354 MB; it expands to
  ~1.9 GB on disk, plus working memory for Neo4j itself). This has been
  tested successfully with Docker Desktop configured for as little as
  4 GB total VM memory on an Apple Silicon Mac.
- No prior Neo4j experience required — this guide assumes none

## Setup (one-time)

1. **Download** `mopek_neo4j_data.tar.gz` from the Zenodo record linked
   in the paper (DOI in the Data and Code Availability section).

2. **Extract it** into this folder, so you end up with a `data/`
   subfolder sitting right next to this README and the
   `docker-compose.yml` file:

   ```bash
   tar -xzvf mopek_neo4j_data.tar.gz
   ```

   After this step, you should see a folder structure like:
   ```
   mopek/
   ├── docker-compose.yml
   ├── README.md
   └── data/
       └── databases/
           └── neo4j/
               └── ...
   ```

3. **Start the database:**

   ```bash
   docker compose up -d
   ```

   The first start takes 30–60 seconds while Neo4j initializes. You can
   watch progress with:

   ```bash
   docker compose logs -f
   ```
   (Press Ctrl+C to stop watching — this does not stop the database.)

## Using the graph

Once running, open your web browser to:

**http://localhost:7474**

Log in with:
- **Username:** `neo4j`
- **Password:** `mopek2026`

You now have the full, live MOPeK graph to query. A few examples to try
(paste into the query box and press the ▶ run button):

```cypher
// See what's in the graph
MATCH (n) RETURN labels(n)[0] AS entity_type, count(*) AS count
ORDER BY count DESC
```

```cypher
// Reproduce Use Case 1 from the paper: drug repurposing via Abarelix
MATCH (p:Peptide {name: "ABARELIX"})-[:BINDS]->(t:Target)
MATCH (t)-[:ASSOCIATED_WITH_DISEASE]->(d:Disease)
RETURN p.name, t.gene_symbol, d.name
```

```cypher
// Look up any peptide by name
MATCH (p:Peptide) WHERE toLower(p.name) CONTAINS toLower("insulin")
RETURN p.peptide_id, p.name, p.source LIMIT 25
```

## Stopping and restarting

To stop the database (your data is preserved):
```bash
docker compose stop
```

To start it again later:
```bash
docker compose start
```

To remove the container entirely (your `data/` folder on disk is still
safe and untouched):
```bash
docker compose down
```

## Changing the default password

The default password (`mopek2026`) is only meant to get you started
quickly. If you plan to run this long-term or on a shared machine,
change it after first login via the Neo4j Browser's user menu, or by
editing `NEO4J_AUTH` in `docker-compose.yml` before first start.

## Troubleshooting

- **"port is already allocated"** — something else on your machine is
  already using port 7474 or 7687. Either stop that other service, or
  edit the `ports:` section in `docker-compose.yml` to map to different
  local ports (e.g. `"7475:7474"`).
- **Neo4j Browser loads but shows no data** — double-check that the
  `data/` folder extracted correctly and sits directly next to
  `docker-compose.yml` (not nested inside an extra folder from the
  archive).
- **"Invalid memory configuration - exceeds physical memory" in
  `docker compose logs`** — this means Docker's own VM has less memory
  available than the values set in `docker-compose.yml`. Open Docker
  Desktop → Settings → Resources and check how much memory is actually
  allocated, then lower the `NEO4J_server_memory_*` values in
  `docker-compose.yml` accordingly (for example, `128m` for each of the
  three settings works on very constrained machines, though performance
  will be slower). After editing, run `docker compose down` followed by
  `docker compose up -d` to apply the change.

## Citation

If you use MOPeK in your research, please cite:

Nallabolu V, Lei [surname]. MOPeK: A Multi-Omics Peptide Knowledge Graph
for Therapeutic Peptide Discovery and Multi-Hop Biomedical Reasoning.
[bioRxiv preprint details].

## Questions

Contact [corresponding author email] for questions, bug reports, or
collaboration inquiries.
