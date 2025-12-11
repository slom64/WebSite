prevents alot of credentials theft, Protected User accounts that authenticate to a domain running Windows Server are unable to do the following:
- Authenticate with NTLM authentication.
- Use DES or RC4 encryption types in Kerberos preauthentication.
- Delegate with unconstrained or constrained delegation.
- Renew Kerberos TGTs beyond their initial four-hour lifetime.

So, Users inside this group may use AES encryption key, you can use this tool to generate the AES key BUT you need to have the plain password, https://github.com/Tw1sm/aesKrbKeyGen.
you can use the AES key like this: `getTGT.py -aeskey <key> -dc-ip "$IP" "$DOMAIN"/"$USER":"$PASSWORD"`


> [!Attention] 
>  When you try to get the AES for computer you should specify `-host` because computers have different salt other than the users salt to generate the right key.
