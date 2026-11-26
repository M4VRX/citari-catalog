# Citari Offline Knowledge Catalog

Public mirror and Ed25519-signed catalog for **Citari** offline knowledge packs *(the Citari Android app is coming soon!)*.

Citari packs are **text-only, unencrypted, and openly inspectable** bundles built from commercially permissive open-knowledge sources (starting with Wikimedia projects under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)). Every pack is cryptographically signed offline so the Citari Android app—and anyone using these files independently—can verify integrity before opening them.

## Mirrors (Active & Planned)

| Priority | Mirror | Status | Location |
| :--- | :--- | :--- | :--- |
| 1 (Planned) | **Cloudflare R2 CDN** | Planned post-launch | Dedicated global CDN primary mirror |
| 1 (Current) | **Hugging Face Datasets** | Active (Primary) | [M4VRX/citari-catalog](https://hf.co/datasets/M4VRX/citari-catalog) |
| 2 (Current) | **GitHub Releases** | Active (Backup, `<= 2 GiB`/file) | [M4VRX/citari-catalog/releases](https://github.com/M4VRX/citari-catalog/releases) |

---

## Philosophy & Open Access

* **Zero DRM / Open Format:** Every pack is a standard SQLite 3 database (`content.sqlite`) paired with a [USearch](https://github.com/unum-cloud/usearch) HNSW vector index (`vectors.usearch`). You do not need the Citari app to read, query, or redistribute these packs under their upstream licenses.
* **Text-Only & Compact:** Articles are stored as clean UTF-8 Markdown compressed with a per-pack trained Zstd dictionary (`pack_blobs`), keeping downloads fast and storage lightweight on mobile devices.
* **Commercial-Safe Sanitization:** Every build runs a fail-closed license gate and Pass 1 sanitizer that strips non-free/Fair Use blocks (`<poem>`, `<score>`, lengthy verbatim quotes) and rejects articles flagged for copyright issues before indexing.
* **Per-Article Attribution:** Every article preserves its upstream `source_url` and `revision_id` inside `content.sqlite`, along with `license_id`, `license_url`, `snapshot_date`, and `modifications_notice` in `pack_meta`.

---

## Currently Published Packs

The live signed catalog index (`catalog.json` + `catalog.sig`) is published under the [`catalog` release](https://github.com/M4VRX/citari-catalog/releases/tag/catalog).

| Pack ID | Title | Upstream Source | Articles | Chunks | Total Size | License | Download |
| :--- | :--- | :--- | ---: | ---: | ---: | :--- | :--- |
| `en-wiki-top50k` | Simple English Encyclopedia | Simple English Wikipedia | 36,937 | 132,704 | 310.9 MB | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) | [GH Release](https://github.com/M4VRX/citari-catalog/releases/tag/pack-en-wiki-top50k) · [HF Mirror](https://hf.co/datasets/M4VRX/citari-catalog) |
| `en-wikibooks-guides` | Wikibooks Complete Library | English Wikibooks | 73,615 | 555,723 | 1.47 GB | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) | [GH Release](https://github.com/M4VRX/citari-catalog/releases/tag/pack-en-wikibooks-guides) · [HF Mirror](https://hf.co/datasets/M4VRX/citari-catalog) |
| `en-wikiversity-cs` | Wikiversity Complete Courses | English Wikiversity | 37,810 | 320,468 | 794.4 MB | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) | [GH Release](https://github.com/M4VRX/citari-catalog/releases/tag/pack-en-wikiversity-cs) · [HF Mirror](https://hf.co/datasets/M4VRX/citari-catalog) |

> **Future Packs:** Additional open-licensed packs (such as Wikivoyage, Appropedia, sharded English Wikipedia, MDN Web Docs, and U.S. Federal Public Domain manuals) will be added to `catalog.json` over time. Always check a pack's `manifest.json` and `pack_meta` table for its exact `license_id`, `license_url`, `snapshot_date`, and disclaimer flags.

---

## Release Structure (GitHub Releases)

Each pack is published under its own release tag (`pack-<pack_id>`) containing **4 core files** (optional `.zstdelta` binary patches may be added for incremental updates):

| File | Description |
| :--- | :--- |
| `content.sqlite` | Read-only SQLite 3 database (`journal_mode=DELETE`, `page_size=4096`) containing `pack_meta`, `pack_blobs` (trained 112 KB Zstd dictionary), `articles`, `chunks` (with exact UTF-16 character spans), `chunks_fts` (FTS5 full-text index), `article_links`, and `article_link_spans`. |
| `vectors.usearch` | USearch HNSW vector index (`i8` cosine or `b1` Hamming) built with `BAAI/bge-small-en-v1.5` (384-dim, CLS pooled, L2 normalized), keyed 1:1 to `chunks.chunk_id`. |
| `manifest.json` | Canonical sorted-key JSON listing `pack_id`, `source`, `snapshot_date`, `license_id`, `article_count`, `chunk_count`, file sizes, SHA-256 digests, and `non_affiliation_notice`. |
| `manifest.sig` | Detached 64-byte Ed25519 signature over the exact bytes of `manifest.json`. |

*(Note: GitHub automatically attaches `Source code (zip)` and `Source code (tar.gz)` to every release tag. Those archives only contain this repository's documentation files and are not part of the knowledge packs.)*

---

## How to Use These Files

### 1. In the Citari Android App *(Coming Soon)*

Once Citari launches on Android, you won't need to download anything manually. The app automatically fetches `catalog.json` and `catalog.sig`, verifies the Ed25519 signature against the pinned public key, and downloads packs from the fastest available mirror with automatic SHA-256 verification.

### 2. Standalone Download (GitHub Releases)

Download the 4 pack files directly from any `pack-<pack_id>` release page or via the GitHub CLI:

```bash
gh release download pack-en-wiki-top50k --repo M4VRX/citari-catalog --dir ./en-wiki-top50k
```

### 3. Verify Ed25519 Signature & SHA-256 Digests

Every `catalog.json` and `manifest.json` is signed with Citari's offline production Ed25519 key:

* **Ed25519 Public Key (hex):** `70521d8b226199d5ea8f4c770f233369c1a03199f8a0d4cda0467ba6189298b3`
* **Public Key SHA-256 Fingerprint:** `7ef85bdab74d002a14abce50b0a3ccc6965e8d1e4d94327db715fafa72fa884b`

Verify a downloaded pack folder in Python (`pip install pynacl`):

```python
import hashlib, json
from pathlib import Path
from nacl.signing import VerifyKey

PUBKEY_HEX = "70521d8b226199d5ea8f4c770f233369c1a03199f8a0d4cda0467ba6189298b3"
pack_dir = Path("en-wiki-top50k")

manifest_bytes = (pack_dir / "manifest.json").read_bytes()
sig_bytes = (pack_dir / "manifest.sig").read_bytes()

# 1. Verify Ed25519 signature
VerifyKey(bytes.fromhex(PUBKEY_HEX)).verify(manifest_bytes, sig_bytes)

# 2. Verify SHA-256 digests for content.sqlite and vectors.usearch
manifest = json.loads(manifest_bytes.decode("utf-8"))
for fname, info in manifest["files"].items():
    digest = hashlib.sha256((pack_dir / fname).read_bytes()).hexdigest()
    assert digest == info["sha256"], f"SHA-256 mismatch for {fname}"

print(f"Verified {manifest['pack_id']} ({manifest['article_count']:,} articles)")
```

### 4. Reading & Searching `content.sqlite`

Article bodies in `articles.body_zstd` are compressed with the Zstd dictionary stored in `pack_blobs` (`key = 'zstd_dict'`). Full-text search works directly via SQLite FTS5 (`chunks_fts`).

Example in Python (`pip install zstandard`):

```python
import sqlite3
import zstandard as zstd

conn = sqlite3.connect("file:en-wiki-top50k/content.sqlite?mode=ro", uri=True)

# Load pack metadata and Zstd dictionary
meta = dict(conn.execute("SELECT key, value FROM pack_meta").fetchall())
zdict = zstd.ZstdCompressionDict(
    conn.execute("SELECT data FROM pack_blobs WHERE key = 'zstd_dict'").fetchone()[0]
)
dctx = zstd.ZstdDecompressor(dict_data=zdict)

# Full-text search using FTS5
rows = conn.execute(
    """
    SELECT c.article_id, a.title, a.source_url, c.section_heading, c.chunk_text
    FROM chunks_fts f
    JOIN chunks c ON c.chunk_id = f.rowid
    JOIN articles a ON a.article_id = c.article_id
    WHERE chunks_fts MATCH ?
    LIMIT 3
    """,
    ("photosynthesis",),
).fetchall()

for art_id, title, url, section, snippet in rows:
    print(f"[{title} — {section}] ({url})\n{snippet[:240]}...\n")

# Decompress a full article body
body_zstd = conn.execute(
    "SELECT body_zstd FROM articles WHERE article_id = ?", (rows[0][0],)
).fetchone()[0]
full_markdown = dctx.decompress(body_zstd).decode("utf-8")
```

### 5. Querying `vectors.usearch`

`vectors.usearch` is a standard [USearch](https://github.com/unum-cloud/usearch) index (384 dimensions, `MetricKind.Cos` + `ScalarKind.I8` for starter packs, or `MetricKind.Hamming` + `ScalarKind.B1` for large shards). Embed your query with `BAAI/bge-small-en-v1.5` (CLS pooling, L2 normalized, empty query prefix), open the index in zero-copy mode via `usearch.index.Index.restore("vectors.usearch", view=True)`, and look up the returned keys directly in `chunks.chunk_id`.

---

## Modifications & Licensing

All packs in this repository are derived from open-licensed upstream dumps (see [LICENSE](LICENSE) for full terms).

* **Summary of Modifications:** Source wikitext/documents are converted to plain UTF-8 Markdown; infoboxes, tables, galleries, images, reference footnotes, stub articles below the pack's character threshold, and non-free/Fair Use blocks (`<poem>`, `<score>`, lengthy verbatim quotes) are removed; text is segmented into UTF-16 citation chunks and compressed with Zstd for offline search.
* **Non-Affiliation:** Independent offline knowledge pack built by Citari. Not affiliated with, sponsored by, or endorsed by the Wikimedia Foundation, the U.S. Government, or any listed source. All trademarks remain the property of their respective owners.
* **Licensing & Takedown Inquiries:** If you spot a licensing, attribution, or copyright concern in any published pack, please open an issue on this repository (or contact the maintainer via the legal contact listed in the Citari app's *Settings → Licenses* screen) with the `pack_id` and article title so it can be audited and removed promptly.
