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

### 3. Submit

The easiest way: after a local run, `pubrun bench` **offers to submit for you** (the prompt
defaults to No — it never transmits without an explicit yes). If you agree, it uses the GitHub
CLI (`gh`) if available, else the GitHub Issues API, else prints a ready-to-paste submission.
Declined it? Submit later without re-running:

```bash
pubrun bench --submit-file <host>-<timestamp>.redacted.json
```

Either way it lands as an **issue** here, so it needs a GitHub account (GitHub has no
anonymous-issue mechanism). Or submit fully manually — attach the **`.redacted.json`** file to
a new issue:

**<https://github.com/fariello/pubrun-benchmarks/issues/new>**

Opening the issue from your own GitHub account lets maintainers follow up with questions —
without any personal data ever being in the file. Fully anonymous submission (a throwaway
account) is fine too.

pubrun **never auto-transmits an un-redacted result**: the submit path verifies the file looks
redacted (no hostname/username/home-path leak) and refuses otherwise.

---

## What's in a result

Each result is a JSON document (current schema: `pubrun-benchmark/4`) with:

- **`machine`** — CPU/GPU model, core count, OS, Python version and implementation,
  environment kind (venv/conda/system), filesystem type of the working directory, the Python
  install, `/dev/shm`, and the I/O-baseline target (the "installed over NFS" signal), and, on
  clusters, Slurm context (partition, node counts). On a `pubrun bench` run each filesystem
  entry also carries a `live` block (free space, inodes, and a hung/slow health status from a
  background probe).
- **`mode`** — the run tier: `quick` (2 measured passes × 15 iterations), `default` (3 × 30),
  `rigorous` (5 × 50), or `custom`. Also `iterations`, `passes`, and `baseline_pass`.
- **`baseline`** — an **uncaptured baseline pass** (the workload run *without* pubrun) that
  warms caches and records the pubrun-absent cost floor. Kept separate (`uncaptured: true`) so
  it is never mixed into the pubrun-overhead statistics.
- **`pass_results`** — the harness runs the full scenario set **N times** (per `mode`) and
  reports each pass, so startup and filesystem-caching effects are visible rather than averaged
  away. Each pass records its own `pass_env`.
- **`scenarios`** — the last (warmest) measured pass mirrored at top level: per-scenario **raw
  timings** (every iteration, in run order) plus summary stats, comparing a baseline run
  against the same workload under pubrun, including ground-truth I/O baselines (`/dev/null`,
  `/dev/shm`, and a temp directory) that isolate filesystem overhead from pubrun overhead. The
  raw samples let anyone recompute any statistic and pool results correctly across machines.
- **`total_wall_time_s`** — wall-clock time for the whole benchmark invocation (harness start
  → end), distinct from the summed per-iteration timings.

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
