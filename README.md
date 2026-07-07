# pubrun-benchmarks

Community-contributed benchmark results for [**pubrun**](https://github.com/fariello/pubrun),
the zero-dependency execution-provenance library for Python research.

pubrun's core promise is that it **never meaningfully intrudes on, slows, or breaks the
host script**. This repository is where that promise is verified in the open: a place to
collect real-world overhead measurements from a wide range of machines, filesystems, and
HPC environments, so the observed cost of running under pubrun is transparent and
reproducible rather than a marketing claim.

> This repo stores and accepts benchmark **data** only. The benchmark tooling lives in the
> main [pubrun repository](https://github.com/fariello/pubrun) under `benchmarks/`.

---

## Contributing a result

You don't need to clone this repo. Three steps:

### 1. Run the benchmark

From a **source checkout** of pubrun (the benchmark harness is intentionally not shipped in
the PyPI wheel, to keep every user install zero-footprint):

```bash
git clone https://github.com/fariello/pubrun.git
cd pubrun
pip install -e .
pubrun bench            # quick sweep; use --full for the complete suite
```

On an HPC login node, `pubrun bench` auto-detects Slurm and offers to submit the run to a
compute node (it only submits after you confirm). See the
[HPC guide](https://github.com/fariello/pubrun/blob/main/docs/hpc.md) for details.

### 2. Find the redacted result

`pubrun bench` writes two files and prints their paths:

| File | Contents | Share it? |
| :--- | :--- | :---: |
| `<host>-<timestamp>.json` | The **full** result — for your own analysis. | No |
| `<host>-<timestamp>.redacted.json` | A **redacted** copy safe to publish. | **Yes** |

### 3. Open an issue

Attach the **`.redacted.json`** file to a new issue:

**<https://github.com/fariello/pubrun-benchmarks/issues/new>**

Opening the issue from your own GitHub account lets maintainers follow up with questions —
without any personal data ever being in the file. Fully anonymous submission (a throwaway
account) is fine too.

---

## What's in a result

Each result is a JSON document (current schema: `pubrun-benchmark/3`) with:

- **`machine`** — CPU/GPU model, core count, OS, Python version and implementation,
  filesystem type of the working directory (the "installed over NFS" signal), and, on
  clusters, Slurm context (partition, node counts).
- **`pass_results`** — the harness runs the full scenario set **twice** and reports each
  pass, so startup and filesystem-caching effects are visible rather than averaged away.
  Each pass records its own `pass_env`.
- **`scenarios`** — per-scenario timings comparing a baseline run against the same workload
  under pubrun, including ground-truth I/O baselines (`/dev/null`, `/dev/shm`, and a temp
  directory) that isolate filesystem overhead from pubrun overhead.

### What is redacted

The `.redacted.json` copy is produced by pubrun's own redactor before you ever see it. It:

- **Masks** hostname, OS username, and every path field that can embed your home directory
  or username (executable paths, prefixes, `virtual_env`, `sys_path`, `source_files`,
  `run_dir`, mount points, tmpdir, …), then does a deep substring scrub of your home-dir
  prefix and username as a safety net.
- **Preserves** the analysis-relevant, non-identifying data: CPU/GPU model, core count,
  timings, versions, filesystem-type classification, and Slurm partition name.

### Privacy caveat (honest)

Even after redaction, a distinctive CPU/GPU model combined with a named Slurm partition can
be re-identifying within a small group. Share only what you are comfortable making public.
Inspect the `.redacted.json` file before attaching it — you can always edit or remove fields
by hand.

---

## License

Contributed benchmark data and the contents of this repository are released under the
**Apache License 2.0** (see [`LICENSE`](LICENSE)), consistent with pubrun itself. By opening
an issue or pull request with benchmark data, you agree it may be published under that
license.

pubrun is copyright 2007–2026 Gabriele G. R. Fariello.
