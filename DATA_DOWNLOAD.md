# RNA expression dataset

`RNA_Seq_All_TPM.csv` is distributed separately through Google Drive because it exceeds GitHub's per-file Git LFS limits. The original CSV remains unchanged.

[Download RNA_Seq_All_TPM.csv from Google Drive](https://drive.google.com/file/d/1QSCQ4nfapLh-337rl7_9Csk7JkEHr4_x/view?usp=sharing).

Open the link and use Google Drive's **Download** button. Download the original CSV, then move it into `data/raw/` without renaming it. The link supports access without signing in; a preview is not required to download the file.

After downloading, place the file at this exact path relative to the repository root:

```text
data/raw/RNA_Seq_All_TPM.csv
```

Expected size: **6,101,790,120 bytes**.

Expected SHA-256:

```text
a6a0ea172b98ef2be08faf6ee8595f8b3629f5e6a31fc79c621a9a8619131b93
```

From the repository root, verify the download on macOS:

```bash
shasum -a 256 data/raw/RNA_Seq_All_TPM.csv
```

On Linux, use `sha256sum` instead of `shasum -a 256`. On Windows PowerShell:

```powershell
Get-FileHash data/raw/RNA_Seq_All_TPM.csv -Algorithm SHA256
```

The downloaded CSV is ignored by Git so it can remain at the notebook's expected path without being accidentally committed. The two other large matrices remain in Git LFS; retrieve them with `git lfs pull`.

This download supplies one training input. The other missing inputs and notebook execution blockers documented in README.md still apply.
