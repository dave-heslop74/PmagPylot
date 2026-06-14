# Data Loading Conventions

This document describes how to load a dataset from the MagIC database given a contribution ID, and the structure of the resulting data.

## Retrieving a contribution from MagIC

MagIC contributions can be downloaded via the EarthRef API:

```
https://api.earthref.org/v1/MagIC/contribution/{ID}
```

This returns the contribution as a tab-delimited text file containing multiple tables (e.g. `sites`, `samples`, `specimens`, `measurements`), each preceded by a header line indicating the table name.

### Example: loading contribution 16663

```python
import requests
import pmagpy.pmag as pmag
import pmagpy.ipmag as ipmag
import pandas as pd

contribution_id = "16663"

url = f"https://api.earthref.org/v1/MagIC/contribution/{contribution_id}"
r = requests.get(url, headers={'Accept': 'text/plain'})

with open('magic_contribution.txt', 'w') as f:
    f.write(r.text)

# Unpack into individual MagIC-format tables
ipmag.download_magic('magic_contribution.txt', dir_path='./data/')
```

This produces individual files such as `./data/sites.txt`, `./data/samples.txt`, etc., each of which can be read with pandas:

```python
sites = pd.read_csv('./data/sites.txt', sep='\t', skiprows=1)
```

The `skiprows=1` skips the MagIC table-type header line (e.g. `tab	sites`).

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
