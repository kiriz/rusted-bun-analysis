# rusted-bun-analysis

Forensic analysis of [oven-sh/bun PR #30412](https://github.com/oven-sh/bun/pull/30412) — the 1M-line Rust rewrite of Bun attributed to Claude Code.

Published as a GitHub Pages site. Analysis scripts in this repo are reproducible against the public bun repository.

## Reproduce

```bash
git clone --filter=blob:none --no-checkout https://github.com/oven-sh/bun.git bun-repo
git -C bun-repo fetch origin pull/30412/head:pr-30412
git -C bun-repo log pr-30412 ^0d9b296af33f2b851fcbf4df3e9ec89751734ba4 \
  --format='%H|||%an|||%ae|||%aI|||%cn|||%ce|||%cI|||%s|||%P' \
  -- > all_commits_git.tsv

python3 analyze_bun_pr.py --mode tsv --tsv all_commits_git.tsv
python3 git_forensics.py --tsv all_commits_git.tsv --repo bun-repo --branch pr-30412 --base 0d9b296af33f2b851fcbf4df3e9ec89751734ba4
```
