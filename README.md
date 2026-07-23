# Legacy Ford TSO Service Manual Conversion Toolkit

Tools for converting lawfully obtained, user-supplied legacy Ford TSO DVD
service manuals into a local static HTML site.

This repository contains conversion tooling only. It does not contain
Ford service information, source archives, extracted content, diagrams,
PDFs, screenshots, or generated sites. **You must supply your own source
media — a physical disc or a digital backup you are entitled to use.**

## Legal Boundary

Use these tools only with source material you have the right to process. Do not
publish or commit generated output from proprietary source material unless you
have explicit permission to do so.

This project is independent and is not affiliated with, sponsored by, or
endorsed by any vehicle manufacturer, service-information publisher, or tool
vendor. Product names, file formats, and path names may appear only where needed
to describe interoperability with user-supplied local files.

## What's Included

```
tools/             Python conversion and verification tools
tests/             pytest coverage for byte-level SVG repair behavior
tso_convert.bat    Windows batch runner for the full conversion flow
pytest.ini         test configuration
```

Generated site files such as `index.html`, `site.css`, `vol_*/`,
`coverage.json`, `catalog.json`, `inventory.json`, `v1_names.json`, and decoded
`content/` trees are local artifacts. They are ignored by Git and should stay
out of public commits.

## Requirements

- Python 3
- A Windows shell for `tso_convert.bat`
- No third-party Python packages are required for the conversion tools

## Quick Start

Run from the repository root in a normal Windows shell.

```
tso_convert.bat <source_dir> <vol_name> ["Display Title" ["Release date"]]
tso_convert.bat finalize
```

`source_dir` must point at a local source root (the DVD or digital backup folder)
that contains the expected `data\` and `content\useni4\` subdirectories.
`vol_name` is the local output directory for that source and should use a
`vol_*` prefix, for example: `vol_05_06_Feb_2007`.

Example:

```
tso_convert.bat D:\ vol_05_06_Feb_2007 "Service Information 2005-2006" "February 2007"
tso_convert.bat finalize
```

The first command extracts and indexes one source folder. This is the quick
part and usually takes only a few minutes. Run it once per source (that is,
once per TSO release).

Once every source has been extracted, run `tso_convert.bat finalize`. This
builds shared metadata, links multiple extracted volumes together when present,
generates the local HTML site (`index.html`) from which you can browse one or
many volumes, rewrites legacy links, repairs SVG compatibility issues (which
previously left some wiring diagrams blank, with no connectors or labels), and
verifies that navigation is browseable. NOTE: This can take a while. 

Set `TSO_PYTHON` to choose a Python executable. Set `TSO_JOBS` to control the
number of parallel extraction workers — raise it to use more CPU cores and
speed up extraction.

## Manual Pipeline

For a single output volume, the batch runner expands to:

```
python tools\inventory.py <source>\content\useni4 --out vol_example\inventory.json
python tools\extract_all.py <source>\content\useni4 --out vol_example\content
python tools\build_coverage.py --root . --vol vol_example --data <source>\data
python tools\recover_v1_names.py
python tools\build_catalog.py --root .
python tools\build_wiring.py
python tools\build_site.py --root .
python tools\rewrite_links.py
python tools\fix_svg.py
python tools\verify_links.py
```

## Development

Run tests from the repository root:

```
python -m pytest
```

Before publishing or opening a pull request, check that only tool, test, and
documentation files are tracked:

```
git ls-files
git status --short --ignored
python -m pytest
```

Do not add source archives, source databases, screenshots, generated sites, or
extracted files to this repository.
