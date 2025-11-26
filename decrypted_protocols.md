# SMB3/HTTPS Forensic Decryption



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

After a failed attempt at brute forcing the hash for user 'eshellstrop'
But then we can use a python script that:
