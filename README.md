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
Create a directory and name it "integrity_checker" 
Inside
Move to the directory "integrity_checker
