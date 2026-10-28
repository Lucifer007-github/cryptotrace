# CryptoTrace

**A Cryptographically-Verified Digital Crime Scene Reconstruction Framework**

Final-year project, Department of Computer Science in Cyber Security, GM University, Davanagere.

| | |
|---|---|
| Team | Deepak J M (U23E01CY007), Krupakar S B (U23E01CY013), Manohar S Naik (U23E01CY015), Nandivardhan U A (U23E01CY019) |
| Guide | Dr. Aruna Kumar B T |
| Status | Review 2 (progress): integrity-and-custody core and verifier working, about 40 percent complete |

## What it does

CryptoTrace makes digital evidence tamper-evident. It:

1. fingerprints every evidence file with SHA-256,
2. combines the fingerprints into a single Merkle root,
3. signs the root with an investigator's private key (ECDSA, curve P-256),
4. records the signed seal in a hash-chained, append-only custody ledger (SQLite), and
5. later re-checks the evidence and reports VERIFIED or TAMPER FLAGGED, naming every MODIFIED, ADDED or DELETED file.

## Architecture

Five layers: Acquisition, Integrity and Custody, Timeline, Verification, Reporting.

| Module | Layer | Status |
|---|---|---|
| `src/hash_evidence.py` | Integrity | Done |
| `src/merkle_builder.py` | Integrity | Done |
| `src/custody_sign.py` | Integrity | Done |
| `src/custody_log.py` | Integrity | Done |
| `src/verify_integrity.py` | Verification | Done |
| `src/acquire.py` | Acquisition | Planned |
| `src/meta_extract.py` | Timeline | Planned |
| `src/timeline_build.py` | Timeline | Planned |
| `src/report_gen.py` | Reporting | Planned |

Helpers: `src/keygen.py`, `src/crypto_utils.py`.

## Folder structure

```
CryptoTrace/
  src/        source code
  tests/      tamper tests (tamper_ledger.py)
  evidence/   sample evidence files
  output/     hashes.json, merkle_root.json, signed_entry.json, verdict.json
  ledger/     custody.db (SQLite)
  keys/       private_key.pem, public_key.pem (never committed)
```

## Setup (Windows PowerShell)

```powershell
git clone <your-repo-url>
cd CryptoTrace
python -m venv venv
venv\Scripts\Activate.ps1
pip install -r requirements.txt
python src\keygen.py --no-password
```

`keys/` is listed in `.gitignore`. Never commit `private_key.pem`.

## Usage

```powershell
python src\hash_evidence.py
python src\merkle_builder.py
python src\custody_sign.py --investigator "Deepak J M"
python src\custody_log.py append
python src\custody_sign.py --investigator "Manohar S Naik" --action HANDOFF
python src\custody_log.py append
python src\custody_log.py show
python src\custody_log.py verify
python src\verify_integrity.py
```

Expected result on untouched evidence: `VERDICT: VERIFIED`.

Do not run `hash_evidence.py` again after sealing. It overwrites the baseline.

## Tamper demonstration

1. Edit, add or delete a file in `evidence/`, then run `python src\verify_integrity.py`. The verdict is `TAMPER FLAGGED` and the file is named.
2. Run `python tests\tamper_ledger.py`, then `python src\custody_log.py verify`. The edit to the ledger is detected.

## How verification works

`verify_integrity.py` runs three checks:

1. Ledger signatures and the hash chain are valid.
2. The baseline hashes rebuild the sealed Merkle root.
3. The current evidence matches the sealed Merkle root.

The result is written to `output/verdict.json`. The exit code is 1 when tampering is flagged.

## Limitations

- It makes tampering detectable, not impossible.
- It does not prove that evidence was genuine before acquisition.
- A stolen private key would allow false signatures. Phase 1 uses one key pair for one investigator. PKI, multi-investigator keys, revocation and rotation are future work.
- It is not a blockchain.
- Removing only the newest ledger entries, or deleting the whole database, is not caught by the chain alone.
- It does not replace Autopsy, FTK Imager or Plaso. It is designed to seal and verify their inputs and outputs.

## Remaining work

Acquisition, metadata extraction, timeline engine with timestamp-consistency flags, PDF case report, a staged incident in a virtual machine with cross-checks against Autopsy and Plaso, and an optional simple GUI.

## Tech stack

Python 3, hashlib, cryptography, SQLite, reportlab, Git. Planned: VirtualBox (Ubuntu), Autopsy / The Sleuth Kit, FTK Imager, Plaso.

## Note

Academic project. Use only with test data. Do not use with real case evidence.
