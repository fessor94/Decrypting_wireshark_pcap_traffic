# SMB3/HTTPS Forensic Decryption

## Cracking NT hash to decrypt **NTLMSSP** traffic on wireshark

#### `> pypykatz lsa minidump Documents/Roomz/Block/evidence-1697996360986/lsass.DMP -o ./dmp.txt`
<img width="1545" height="780" alt="1" src="https://github.com/user-attachments/assets/460f7f53-061a-476e-8cc6-5cb1d42bca75" />

pypykatz - This calls Pypykatz.
lsa - This tells Pypykatz to use the LSA (Local Security Authority) module. In this context, it's focused on extracting secrets that are stored in LSASS memory.

We can then look and search for the NTLM password hash of 'mrealman':

#### `> cat Documents/TryHackme/Block/evidence-1697996360986/dmp.txt | grep 'mrealman' -A 10`
<img width="893" height="715" alt="5" src="https://github.com/user-attachments/assets/e5047631-2d9a-4eef-9143-8225e60d6a27" />

#### `NT: 1f9175a516211660c7a8143b0f36ab44` 

and use a hash cracking tool:
<img width="1043" height="386" alt="2" src="https://github.com/user-attachments/assets/fd466360-60c4-4da4-a3d7-a3fdb8096d40" />

The result of the NTLM password hash is: **Blockbuster1**
Which we can use to DECRYPT SMB3 traffic assosiated with the user 'mrealman':
On wireshark you can navigate to: 
**Edit > Preferences > Protocols > NTLMSSP**
Once this is decryped I can then be able to go to:
**File > Export Objects > SMB** to download the decrypted traffic

<img width="711" height="500" alt="3" src="https://github.com/user-attachments/assets/10515e2b-e2c0-444d-9646-95107ba1f715" />
<img width="498" height="444" alt="4" src="https://github.com/user-attachments/assets/71245923-34cd-4612-a904-8971cc0a4fbd" />

## Creating A Keytab to decrypt **KRB5** traffic on wireshark

After a failed attempt at brute forcing the hash for user 'eshellstrop'
We can:

**Extract NTLM hash from LSASS memory dump**
`pypykatz lsa minidump Documents/Roomz/Block/evidence-1697996360986/lsass.DMP -o ./dmp.txt`

Filter for the user eshellstrop
`grep -A 10 'eshellstrop' ./dmp.txt `
Get the:
NT: 1f9175a516211660c7a8143b0f36ab44

**Open ktutil on Linux**

ktutil

`ktutil: add_entry -p eshellstrop@BLOCK -k 1 -e rc4-hmac -w 1f9175a516211660c7a8143b0f36ab44`
<img width="832" height="124" alt="1-2" src="https://github.com/user-attachments/assets/0489d2a6-4e50-4544-8236-d47ec8a57f8d" />


````
- `-p eshellstrop@BLOCK` → Kerberos principal
- `-k 1` → Key version number (from the captured ticket)
- `-e rc4-hmac` → Kerberos encryption type
- `-w 1f9175a516211660c7a8143b0f36ab44` → Key value (NT hash)
````

**Behind the scenes:**

- For RC4-HMAC, the NT hash is directly used as the encryption key.
- `ktutil` stores this as a binary entry in memory.

### Verify the keytab entry

```
ktutil: list
````

**Output:**

```
slot KVNO Principal
---- ---- -------------------
   1    1 eshellstrop@BLOCK
```

Confirms the keytab entry is present and ready.

### Write the keytab to disk

```text
ktutil: write_kt myhash.keytab
ktutil: quit
```

* Creates the binary file `myhash.keytab`.
* This file contains the principal, KVNO, encryption type, and the key (NT hash).

### Use in Wireshark

1. Go to Wireshark
2. Navigate to:
   `Edit > Preferences > Protocols > KRB5`
3. Add `myhash.keytab`
<img width="838" height="729" alt="1-3" src="https://github.com/user-attachments/assets/91417000-2035-4e86-9266-3386c715407e" />

Wireshark now attempts to decrypt any RC4-HMAC Kerberos traffic for this principal.

```
```
You can now see eshellstrop traffic decrypted:
<img width="1861" height="556" alt="1-4" src="https://github.com/user-attachments/assets/f5deaf77-9c47-43f0-a357-65f8be0ea7c6" />

You can now navigate to:
File > Export Objects > SMB
<img width="761" height="554" alt="1-5" src="https://github.com/user-attachments/assets/b9d952c9-63b9-4232-8bae-685d2b8c9b6b" />

