# Table of Contents
- [GPG in General](#gpg-in-general)
- [Symmetric vs Asymmetric](#symmetric-vs-asymmetric)
- [Installation](#install)
- [Managing Key Pairs](#managing-key-pairs)
  - [Creating a Key Pair](#creating-a-key-pair)
  - [List all Keys](#list-all-keys)
  - [Deleting Keys](#deleting-keys)
  - [Exporting Keys](#deleting-keys)
  - [Importing Keys](#importing-keys)
  - [Public Key Servers](#public-key-servers)
  - [Editing Keys](#editing-keys)
  - [Revoking a Key](#revoking-a-key)
  - [Signing Public Keys](#signing-public-keys)
- [Encrypting Files](#encrypting-files)
- [tar Archive Guide](#tar-archive-guide)
- [Signing Files](#signing-files)
- [GPG-Agent Cache](#gpg-agent-cache)
- [Subkeys](#subkeys)
- [Handling Private Keys](#handling-private-keys)
- [Backup](#backup)
  - [Paper Backup](#paper-backup)
- [References and Sources](#references-and-sources)

# GPG in General
**GPG** aka **GnuPG** aka **Gnu Privacy Guard**  
**GNU**: a recursive acronym meaning "_**GNU's not Unix!_**")  
  
GPG (OpenPG) is the adaption of the encryption standard known as PGP (Pretty Good Privacy). It is mainly designed to use asymmetric public/private key cryptography but also has the option to symmetrically encrypt data. 

# Symmetric vs Asymmetric
Symmetric key cryptography uses the same key for both encryption and decryption. This means that both parties that want to communicate have to agree on the key beforehand. The main difficulty with symmetric key cryptography is the key exchange itself.

Asymmetric key encryption uses a private / public key pair. The public key is used to encrypt data for you and only the private key, which has to be kept secret, can decrypt the data.

Both schemes are used together for optimal performance. This means using asymmetric encryption to exchange a symmetric key that has been used encrypt data. This is faster because symmetric encryption is a lot faster than asymmetric encryption. This way both parties only need to know the public key of each other to exhange the symmetric key over an encrypted channel. The symmetrically encrypted data can then be sent over unsecured channels (assuming the passphrase is strong enough).

# Install
- Windows: [gpg4win](https://www.gpg4win.org/get-gpg4win.html)  
- MacOS: `brew install gpg pinentry-mac`  
- Linux: `sudo apt-get install gnupg`  

On Windows most of the following functionality can be handled via the installed gpg4win GUI application called Kleopatra.

## Verify Installation
In this guide GnuPG version `2.3.4` was used.
``` shell
gpg --version
```

# Managing Key Pairs
## Creating a Key Pair
```shell
gpg --full-gen-key
```
You will be prompted several things:
1. Keypair kind (RSA & RSA or ECC are the 2 most likely options for you)
2. Keysize (RSA), Elliptic Curve (ECC)
3. Expiration Date for the key (this can be extended manually later on, so picking a key with an expiration is the safest option in case you lose the key)
4. Name and email address
5. Comment
6. Passphrase: Choose a secure password to protect your key

## List All Keys
``` shell
-k = --list-keys
-K = --list-secret-keys

// List public keys only
gpg -k

// List all private keys available
gpg -K

//List all private keys with subkey fingerprints
gpg -K --with-subkey-fingerprints
```

```shell
gpg --fingerprint [KEYID]      //without KEYID supplied it will list all fingerprints
```


## Deleting Keys
```shell
gpg --delete-key KEYID
gpg --delete-secret-key KEYID
```
KEYID can be anything that identifies the key like the name of the owner, email, fingerprint etc.

## Exporting Keys
```shell
-a = --armor (ASCII instead of Binary)
-o = --ouptut

// Public
gpg --export -a -o PUBLIC.asc KEYID

// Private
gpg --export-secret-keys -a -o PRIVATE.asc KEYID
```
Without --output specified gpg will print to stdout

Note: The secret key export includes a full export (because the public key and all secret parts are included in the private secret key)

## Importing Keys
```shell 
// Public (your own or someone elses)
gpg --import PUBLIC.key

// Private
gpg --allow-secret-key-import --import PRIVATE.asc
```

## Public Key Servers 
-   [pgp.mit.edu](http://pgp.mit.edu/)
-   [keyserver.ubuntu.com](https://keyserver.ubuntu.com/)
-   [keyserver.pgp.com](https://keyserver.pgp.com/)
-   [peegeepee.com](https://peegeepee.com/)
- etc.

### Publish Public Key
You can and should publish your public for others to use it. You can either distribute it directly, add it to your website or use a public keyserver.
```shell
gpg --send --keyserver hkp://KEYSERVER KEYID
gpg --send-keys --keyserver hkp://KEYSERVER KEYID
```
### Search Public Keys
```shell
gpg --keyserver hkp://KEYSERVER --search USER@DOMAIN.COM
```

Where KEYSERVER is the address of the key server like: pgp.mit.edu

## Editing Keys
```shell
gpg --edit-key KEYID
```
A interactive gpg prompt will start, get all command by entering `help`. Options include: 
- `trust`: Change trust level
- `expire`: Change expiration date
- `passwd`: Change passphrase
- `adduid`/`deluid`: Add remove user id

uid's are numbered, entering this number in the prompt will select/deselect the uid.

## Revoking a key
If the private key has been compromised or a UID has changed or simply if you forgot the passphrase the public key has no use anymore. It is recommended to generate a revocation certificate immediately after generating a key and keeping the revocation certificate somewhere safe. 
```shell
gpg -o revoke.asc -a --gen-revoke KEYID
```
To revoke the key do:
```shell
gpg --import revoke.asc
```
You can now replace the existing public keys on keyservers with the revoked one to notify people of the change.  Or simply replace the old public key a new one. You have to generate a new keypair for this.

## Signing Public Keys
``` shell
gpg --sign-key recipient@email.com
```
### Show Fingerprint of Public Key
In order to verify the key manually, you can compare the local fingerprint of the public key with a the fingerprint of the same key provided directly and securely by the recipient itself.
``` shell
gpg --fingerprint recipient@email.com
```


# Encrypting Files
## Symmetric
```shell
-c = --symmetric (encrypt)
-o = --output (optional)

gpg -c [--cipher-algo AES256] [-o OUTPUT] INPUT
```
The default symmtric cipher in gpg 2.3 is AES-256 so passing the `--cipher-algo` flag is unnecessary unless you want to use something else or be sure.
Alternatively you can also set `--cipher-algo AES256` in your `gpg.conf` file or in the command line so you don't have to specify it everytime. Or you could edit the cipher preferences on your key with `set-pref` inside the edit menu.

Using AES-256 will use the CFB block cipher mode internally.

## Asymmetric
```shell
-e = --encrypt (asymmetric)
-r = --recipient
-u = --local-user (local key used to encrypt)(optional)
-o = --output (optional)

gpg -e -r KEYID [-u KEYID] [-o OUTPUT] INPUT
```

If you don't supply the `-o` output file flag, gpg will create a new file with the same filename + `.gpg` appended (`.asc` if ascii armored output)

### Multiple Recipients
An additional  feature of public key cryptography is the ability to encrypt for multiple recipients. Therefore in gpg you can specify multiple recipients by adding multiple `-r KEYID` flags.

### Compression
Encryption uses compression by default. To disable, use the option `-z 0`. This will speed up the process if encrypting a large file which is already compressed.

```shell
gpg -e -z 0 -r KEYID file.tar.gz
```

### Listing packets
Lists infos about the gpg file. Can be signed or encrypted data.
```shell
gpg --list-packets GPGFILE
```


## Decrypting Files
This works for both symmetrically and asymmetrically encrypted files
```shell
-d = --decrypt
-o = --output (optional)

gpg -d -o OUTPUT INPUT
or
gpg INPUT (will decrypt and save to filename without .gpg or .asc ending)

```
Not having an output file specified will result in gpg printing to stdout.


## ASCII Armor
Normally encrypted data and exports will be binary. If you intend to send the encrypted data over the internet (email etc.) to someone it has to be in ASCII form. Use the `-a` flag which stands for `--armor` to have an ASCII output.

Simply storing data on a NAS or never having to paste it somewhere doesn't require the `--armor` flag.

Using ASCII armor will results in bigger files and longer encryption times:
> We do not recommend using the --armour option for encrypting files that will be transferred to/from NAS systems. This option is mainly intended for sending binary data through email, not via transfer commands such as bbftp or ftp. The file size tends to be about 33% bigger than without this option, and encrypting the data takes about 10-15% longer.

Quote by [NASA's guide for GPG file encryption](https://www.nas.nasa.gov/hecc/support/kb/using-gpg-to-encrypt-your-data_242.html)


# tar Archive Guide
```shell
-c create
-x extract
-f filename
-z gzip compression
-j bzip compression
-v verbose

// Create .tar archive
tar -cf archive.tar FOLDER/ [optional more files]


// Extract .tar
tar -xf archive.tar [-C extractFolder]

// Create gzipped archives
tar -czf archive.tar.gz FOLDER/


// Extract gzipped
tar -xzf archive.tar.gz


//List Contents:
tar -tvf archive.tar

```


# Signing Files

```shell
-s = --sign
-u = --local-user (local key used to sign)(optional)

// Sign file (binary output)
gpg [-u KEYID] -o OUTPUT -s INPUT 

// Sign file (ASCII output)
gpg [-u KEYID] -o OUTPUT --clear-sign INPUT
```
Both options result in the file being wrapped inside the signature. This means that for non-trivial documents like simple text, the receiver needs to edit the files to extract the original file data without the signature. This can also be achieved by simply decrypting the signed file into a seperate output.

To circumvent this problem you can create a detached signature into a seperate file. This way the receiver can use the signed document as intendet and can verify the validity with the seperate signature file. This is highly recommended when signing binary data (e.g. archives)
```shell
-b = --detach-sign

// Binary output
gpg -o OUTPUT.sig -b INPUT

// ASCII output
gpg -a -o OUTPUT.asc -b INPUT
```
### Sign and encrypt file
```shell
gpg -s -e -r KEYID [-u KEYID] [-a] INPUT
```
This will first sign the file and then encrypt it. When decrypting the signature is automatically checked (assuming the public key of the sender has been imported).

## Verify Signatures
```shell
gpg --verify INPUT
```
INPUT can be any signed file or detached signature.

### Check Signature of downloaded file
```shell
gpg --verify FILE.txt.asc FILE.txt
```


# GPG-Agent Cache
While playing around locally, encrypting and decrypting files you might have noticed that the password for keys gets cached locally for a short time. This of course has the advantage that you don't have to enter the key all the time. The cache gets cleared periodically or when restarting of course.

If you want to clear the cache manually:
```shell
gpg-connect-agent reloadagent /bye
```


# Subkeys
Subkeys make key management easier since your primary key pair is very important and holds a lot of power. Therefore you should keep it very safe, but this safety would also come at a cost when you try to encrypt or sign data. This is where subkeys come in.  

When you created your keypair, gpg automatically created an encryption subkey. To seperate encryption, signing and if you need to authentication into fully seperate subkeys you have to create a new key for each functionality.
The certify functionality shall be kept only on the primary secret key as this enables the signing of new subkeys.

Now you keep only the subkeys on your machine, publish them instead of your primary key and use them instead of the primary key.

You will need access to the secret key only in the following scenarios:
-   sign someone else's key or revoke an existing signature,
-   add a new UID or mark an existing UID as primary,
-   create a new subkey,
-   revoke an existing UID or subkey,
-  change the preferences (e.g., with setpref) on a UID,
-  change the expiration date on your primary key or any of its subkey, or
-   revoke or generate a revocation certificate for the complete key.

## Create subkeys

```shell
gpg --edit-key KEYID addkey
// Go through key creation guidde

gpg> save
```

## Remove private primary key
```shell
// Export all subkeys
gpg -o secret-subkeys --export-secret-subkeys
// Alternatively you can specify a specific subkey with KEYID! (! is important)

//Delete secret primary key
gpg --delete-secret-key KEYID

//Import subkey
gpg --import secret-subkeys
```
`gpg -K` should now show a `sec#` instead of a `sec` for your private key, indicating a missing secret key.
Remember to delete the file containing the subkeys afterwards.  
Additionally you should also change the passphrase protecting your subkeys with:
```shell
gpg --edit-key KEYID passwd
```
This way if your subkey set gets compromised your primary key backup is still protected with the original passphrase.



# Handling Private Keys
Private keys should be kept secret and not be exported or shared.  
Depending on your threat assessement and security needed you have several options options:

From [Jens Erat's Stackexchange Answer](https://security.stackexchange.com/questions/31594/what-is-a-good-general-purpose-gnupg-key-setup)
> Your computer always could be hacked or infected by some malware downloading your keys and installing a key logger to fetch your password (and this is not a matter of which operating system you use, all of them include severe security holes nobody knows about at this time).  

> **Keeping your primary (private) key offline is a good choice** preventing these problems. It includes some hassles, but reduces risks as stated above.    

>Highest security would of course mean to use a separate, offline computer (hardware, no virtual machine!) to do all the key management using your primary key and only transferring OpenPGP data (foreign keys and signatures you issued) using some thumb drive.


# Backup
Export backup from private secret key (including trustdb etc.)
```shell
gpg -o private_backup.gpg -a --export-secret-keys --export-options export-backup KEYID
```
The exported private key is encrypted by default with the passphrase you chose for it when you created the keypair.

## Restore
```shell
gpg --import-options restore --import private_backup.gpg
```

## Paper backup
As an alternative to the following procedure you could also use [Paperkey](https://www.jabberwocky.com/software/paperkey/).  
I just wanted to have qr codes and the extra unnecessary data printed, doesn't bother me much.  

Use linux or wsl for the following steps.

Install packages:
```shell
sudo apt-get install qrencode
```


In order to safely store the created backup file of the secret private key, we can split the generated (ascii armored!) file into 2kB files
```shell
split -b 2000 -d backup.key part
```
Here we split backup.key into multiple files with a max of 2000 bytes. They are numbered like: `part00 part01...`

Next run the following bash script to encode every part as a qr image
```shell
for file in part?? 
do
	qrencode -l M -r $file -o "$file".png
	rm $file	
done
```

Now your can print all the qrcodes on paper in the correct order!
**Don't print the passphrase for the secret key! instead write it on the paper by hand**

### Restore from paper
First scan in, take pictures or whatever. Just get seperate correctly ordered images of the qr codes on your machine.  
Label them `part00.png...part??.png`

Install zbar-tools:
```shell
sudo apt-get install zbar-tools
```
Run this script to read the qr codes with zbarimg and write them concatenated into a new file called `restored_key.gpg`
```shell
for file in part??.png
do
	zbarimg --raw $file > part_${file//[^0-9]/}
done

cat part_* > restored_key.gpg
rm part_*

```

Now you can import the restored secret key like exaplained in [Restore](#restore)

# References and Sources
- [bfrg's gpg guide](https://github.com/bfrg/gpg-guide)
- [samuelexferri's gpg guide](https://github.com/samuelexferri/gpg-guide/blob/master/gpg-guide.md)
- [NASA's gpg encryption guide](https://www.nas.nasa.gov/hecc/support/kb/using-gpg-to-encrypt-your-data_242.html)
- [Debian's Subkey Guide](https://wiki.debian.org/Subkeys?action=show&redirect=subkeys)
- [Andy Gock's GPG Cheatsheet](https://gock.net/blog/2020/gpg-cheat-sheet/)
- [Secret Key Backup (Rubberstamp's Answer on Stackexchange)](https://unix.stackexchange.com/a/482559)
- [Paul Fawkesley - Protect Private Key](https://paul.fawkesley.com/gpg-for-humans-protecting-your-primary-key/)
- [Paul Fawkesley - Prepare Offline Machine](https://paul.fawkesley.com/gpg-for-humans-preparing-an-offline-machine/)
- [Jens Erat on a good gnupg setup](https://security.stackexchange.com/questions/31594/what-is-a-good-general-purpose-gnupg-key-setup)
- [Jens Erat on how many gpg keys to make]( https://security.stackexchange.com/questions/29851/how-many-openpgp-keys-should-i-make)
  