# Data Loading Conventions

This document describes how to load a dataset from the MagIC database given a contribution ID, and the structure of the resulting data.

## Retrieving a contribution from MagIC

**Preferred method: `ipmag.download_magic_from_id()`** — per `functions/policy.md`, prefer this tested PmagPy function over hand-rolled `requests` calls to the MagIC API:

```python
import pmagpy.ipmag as ipmag

success, result = ipmag.download_magic_from_id("19938", directory="./data/")
if success:
    contribution_file = result  # e.g. 'magic_contribution_19938.txt'
else:
    print("Download failed:", result)
```

This downloads the contribution as a tab-delimited text file containing multiple tables (e.g. `sites`, `samples`, `specimens`, `measurements`), each preceded by a header line indicating the table name. Internally, this function makes a request to `https://api.earthref.org/v1/MagIC/data`.

**Note on execution environments:** Some sandboxed code-execution environments (including, at the time of writing, claude.ai's bash/code tool) restrict outbound network access to an allowlist of domains that does not include `api.earthref.org`. `ipmag.download_magic_from_id()` will fail in such environments (typically returning `success=False` with a "Forbidden" error) for exactly the same underlying reason a direct `requests.get()` to the same host would fail — **this is not a bug in the function**, and switching to a manual `requests` call will not help, since it hits the same host. If this happens, do not keep retrying variations of the API call — instead use the alternative method below. In environments with normal network access (e.g. a local install, Tier 1/2 deployments — see `README.md`), this function works as expected.

### Alternative: user downloads and uploads the data directly

If direct API access is unavailable, ask the user to download the contribution from the MagIC web interface (e.g. `https://www2.earthref.org/MagIC/{ID}` or via search results on earthref.org) — this downloads as a zip file containing one or more `magic_contribution_{ID}.txt` files — and upload/attach it to the conversation. This file can then be unzipped and parsed exactly as described below, entirely locally, with no network access required. This is a fully viable primary workflow, not just a fallback — see `README.md` for discussion of deployment tiers.

A single zip may contain multiple contribution files if the user downloaded multiple search results; check which file(s) actually contain a `sites` table (via `tab delimited\tsites` in the file) before proceeding, and confirm with the user if more than one candidate is found.

### Unpacking a contribution file into MagIC-format tables

Once a contribution file is available (whether from `download_magic_from_id` or an uploaded file), `ipmag.download_magic()` (a different, confusingly-similarly-named function — this one is a local file unpacker, not a network downloader) unpacks it into individual table files:

```python
ipmag.download_magic('magic_contribution_19938.txt', dir_path='./data/', input_dir_path='./data/')
```

This produces individual files such as `./data/sites.txt`, `./data/samples.txt`, etc., each of which can be read with pandas:

```python
sites = pd.read_csv('./data/sites.txt', sep='\t', skiprows=1)
```

The `skiprows=1` skips the MagIC table-type header line (e.g. `tab	sites`).

### Example: loading an uploaded contribution file directly

If working from an uploaded `magic_contribution_{ID}.txt` and `ipmag.download_magic()`'s directory conventions are inconvenient (e.g. for a single quick analysis), the `sites` table can be extracted directly by locating the `tab delimited\tsites` marker line and reading from the following line as the table header:

```python
import pandas as pd

with open('magic_contribution_19938.txt') as f:
    lines = f.readlines()

# Find the line announcing the sites table
sites_start = next(i for i, line in enumerate(lines) if line.strip() == 'tab delimited\tsites')

with open('sites_table.txt', 'w') as f:
    f.writelines(lines[sites_start + 1:])

sites = pd.read_csv('sites_table.txt', sep='\t')
```

This is a simple, self-contained parsing step (not a calculation covered by `functions/policy.md`'s "use PmagPy functions" rule) and is the approach used for the worked examples in `community/edge_cases.md`. Prefer `ipmag.download_magic()` when working with a full multi-table contribution and standard directory layouts; this direct approach is convenient for single-file, single-table extraction.

## Filtering for site mean directions

For workflows that operate on site-level mean directions (such as the reversal test), filter the `sites` table for rows with a method code indicating a Fisher mean direction was calculated, e.g. `DE-BFP` (best-fit plane/PCA-derived) or `DE-FM` (Fisher mean):

```python
sites = sites[sites['method_codes'].str.contains('DE-BFP|DE-FM', na=False)]
```

Relevant columns for direction-based workflows typically include:

| Column | Meaning |
|---|---|
| `dir_dec` | Declination of the site mean direction (degrees) |
| `dir_inc` | Inclination of the site mean direction (degrees) |
| `dir_k` | Fisher precision parameter (κ) for the site |
| `dir_alpha95` | 95% confidence cone (α95, degrees) |
| `dir_n_samples` | Number of samples contributing to the site mean |
| `method_codes` | Pipe- or colon-separated list of method codes describing how the direction was derived |

## A note for the LLM on data inspection

Before proceeding with any analysis, briefly summarise what was loaded — number of sites, polarity breakdown, location/study name if available — so the user can confirm the right dataset was loaded. Do not assume the data is "clean"; if columns are missing or contain unexpected values (e.g. all `dir_inc` values positive, suggesting no reversed polarity sites are present), flag this to the user before proceeding with a workflow that depends on it.

## Coordinate systems

MagIC data may be reported in geographic coordinates, tilt-corrected (structural/stratigraphic) coordinates, or both, distinguished by the method codes (`DA-DIR-GEO`, `DA-DIR-TILT`) or by separate columns/rows. Workflows that are sensitive to coordinate system (e.g. fold tests, pole calculations) should check which coordinate system is present and flag this to the user. The reversal test described in this knowledge base is **not** sensitive to coordinate system, since it only depends on the relative directions of normal and reversed populations.
