# Understanding the Process

This document explains what is happening step-by-step when analyzing an LSASS memory dump, extracting an NTLM hash, recovering a password, and using it to decrypt SMB3 network traffic in Wireshark.

---

## 1. Extracting Credentials From an LSASS Memory Dump

Command:

pypykatz lsa minidump Documents/Roomz/Block/evidence-1697996360986/lsass.DMP -o ./dmp.txt


Explanation:

- LSASS is a Windows process that temporarily stores authentication data.
- Pypykatz analyzes the LSASS memory dump and extracts:
  - usernames
  - NTLM password hashes
  - other authentication artifacts
- This step is common in digital forensics when investigating how a user or attacker interacted with a system.

---

## 2. Searching for the NTLM Hash of a Specific User

Command:

cat Documents/TryHackme/Block/evidence-1697996360986/dmp.txt | grep 'mrealman' -A 10


Explanation:

- You search the extracted data for the user `mrealman`.
- This reveals the NTLM hash for that account.

Example extracted hash:

NT: 1f9175a516211660c7a8143b0f36ab44


---

## 3. Recovering the Password From the NTLM Hash

- A hash-cracking tool is used to attempt to recover the original password associated with the NTLM hash.
- The recovered password in this case is:

Blockbuster1


- In a forensic investigation, recovering a password can allow you to decrypt captured traffic or understand user activity.

---

## 4. Decrypting SMB3 Traffic in Wireshark Using the Credentials

With the recovered password, Wireshark can decrypt SMB3 traffic associated with the user.

Navigate to:

Edit > Preferences > Protocols > NTLMSSP


- Enter the recovered password and username information.
- Wireshark will use these credentials to decrypt SMB3 packets.
- This allows encrypted SMB communication to become readable for analysis.

---

## 5. Exporting the Decrypted SMB Objects in Wireshark

Once SMB3 traffic is decrypted, you can extract transferred files.

Navigate to:

File > Export Objects > SMB


- This shows a list of files transmitted during the SMB session.
- You can save these files for further forensic examination.

---


