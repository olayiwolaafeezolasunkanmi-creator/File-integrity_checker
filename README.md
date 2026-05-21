# File-integrity_checker
A file integrity checker is a tool that detect unauthorised changes to file by computing, analyzing and verifying SHA-256 hashes. This  project guides you in buiding a simple and effective tool to monitor highly file integrity and quickly spot tampering or vulnerabilities. 
# SHA-256 
SHA-256 is for Secure Hash Algorithm 256-bit. It’s a type of hashing algorithm used in cybersecurity to turn any data (text, password, file, etc.) into a fixed-length string of characters called a hash.
# Step by step process to solve the problem
Understand input/output
Choose Programming Language (Python for ease)
Define Storage format JSON dictionary (file_path: hash)
Secure location: use os.path.expanduser("~/.integrity_check/hashes.json") and create a directory with permission 0o700.
Implement hashing: read file in chunks for large files.
Implement recursive directory walking.
Implement command parsing using argparse.
Error handling (permission denied, file not found). 
Reporting with colors or simple text.
# Step by step implementation

Create a directory, name it "integrity_checker"
```bash
mkdir integrity_check``
```
change directory into "integrity_check"
```bash
cd integrity_check``
```
nano the hashes inside the directory 
```bash
nano hashes.json
```
create directory  "integrity_tool" 
```bash
mkdir integrity_tool
```
change directory to "integrity_tool"
```bash
cd integrity_tool
```
create new empty file on "integrity_check"
```bash
touch integrity_check
```
give the file permission to execute as a programe 
```bash
chmod +x integrity_check
```
go to directory that stores file 
```bash
./integrity_check init var/log/
```
create a file editor inside "integrity_check"
```bash
nano integrity_check
```
then write your code inside the file editor
```bash
"""

import argparse
import os
import json
import hashlib
from pathlib import Path

# ========== CONFIGURATION ==========
STORAGE_DIR = Path.home() / ".integrity_check"
STORAGE_FILE = STORAGE_DIR / "hashes.json"


# ========== STORAGE HELPERS ==========
def ensure_storage():
    """Create storage directory and file with secure permissions."""
    STORAGE_DIR.mkdir(mode=0o700, exist_ok=True)
    if not STORAGE_FILE.exists():
        STORAGE_FILE.touch(mode=0o600)
        STORAGE_FILE.write_text("{}")


def load_hashes():
    """Load stored hash dictionary from JSON file."""
    ensure_storage()
    with open(STORAGE_FILE, "r") as f:
        return json.load(f)


def save_hashes(hashes):
    """Save hash dictionary to JSON file with secure permissions."""
    ensure_storage()
    with open(STORAGE_FILE, "w") as f:
        json.dump(hashes, f, indent=2)
    os.chmod(STORAGE_FILE, 0o600)


# ========== HASHING UTILITY ==========
def compute_hash(file_path):
    """Return SHA-256 hex digest of a file (reads in chunks for large files)."""
    sha256 = hashlib.sha256()
    with open(file_path, "rb") as f:
        # Read in 64KB chunks to handle large files efficiently
        for chunk in iter(lambda: f.read(65536), b""):
            sha256.update(chunk)
    return sha256.hexdigest()
# ========== FILE COLLECTION ==========
def collect_files(path):
    """
    Return a list of absolute file paths.
    If path is a file, return [path].
    If path is a directory, recursively return all regular files inside.
    """
    path = Path(path).resolve()
    if path.is_file():
        return [str(path)]
    elif path.is_dir():
        files = []
        for root, _, filenames in os.walk(path):
            for f in filenames:
                full = Path(root) / f
                if full.is_file():
                    files.append(str(full))
        return files
    else:
        raise FileNotFoundError(f"Path not found: {path}")


# ========== COMMAND: INIT ==========
def cmd_init(path):
    """Initialize hash database for given path."""
    files = collect_files(path)
    hashes = {}
    for f in files:
        try:
            hashes[f] = compute_hash(f)
            print(f"Hashed: {f}")
        except Exception as e:
            print(f"Error hashing {f}: {e}")
    save_hashes(hashes)
    print(f"Hashes stored successfully for {len(files)} files.")


# ========== COMMAND: CHECK ==========
def cmd_check(path):
    """Check current file hashes against stored hashes."""
    stored = load_hashes()
    files = collect_files(path)

    modified = []
    unmodified = []
    not_initialized = []

    for f in files:
 if f not in stored:
            not_initialized.append(f)
            continue
        try:
            current_hash = compute_hash(f)
            if current_hash == stored[f]:
                unmodified.append(f)
            else:
                modified.append(f)
        except Exception as e:
            print(f"Error reading {f}: {e}")

    # Report results
    for f in modified:
        print(f"Status: Modified - {f}")
    for f in unmodified:
        print(f"Status: Unmodified - {f}")
    for f in not_initialized:
        print(f"Status: Not Initialized - {f}")

    if not modified and not not_initialized:
        print("All checked files are unmodified.")
    elif modified:
        print("Possible tampering detected!")


# ========== COMMAND: UPDATE ==========
def cmd_update(path):
    """Update stored hash for given file(s) to current value."""
    stored = load_hashes()
    files = collect_files(path)

    updated = 0
    for f in files:
        if f not in stored:
            print(f"Skipping {f} – not in stored hashes. Use 'init' first.")
            continue
        try:
            new_hash = compute_hash(f)
            stored[f] = new_hash
            updated += 1
            print(f"Updated: {f}")
        except Exception as e:
            print(f"Error updating {f}: {e}")

    if updated:
        save_hashes(stored)
        print(f"Hash updated successfully for {updated} file(s).")

# ========== MAIN DISPATCHER ==========
def main(): 
    parser = argparse.ArgumentParser(
        description="Log file integrity checker using SHA-256",
        epilog="Example: ./integrity-check init /var/log"
    )
    subparsers = parser.add_subparsers(dest="command", required=True)

    # init command
    init_parser = subparsers.add_parser("init", help="Initialize hash database")
    init_parser.add_argument("path", help="File or directory to initialize")

    # check command
    check_parser = subparsers.add_parser("check", help="Check integrity against stored hashes")
    check_parser.add_argument("path", help="File or directory to check")

    # update command
    update_parser = subparsers.add_parser("update", help="Update stored hash for a file")
    update_parser.add_argument("path", help="File or directory to update")

    args = parser.parse_args()

    if args.command == "init":
        cmd_init(args.path)
    elif args.command == "check":
        cmd_check(args.path)
    elif args.command == "update":
        cmd_update(args.path)


if _name_ == "_main_":
    main()
```
