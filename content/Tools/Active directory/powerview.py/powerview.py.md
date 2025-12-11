
### Starting web interface
```
powerview "$DOMAIN"/"$USER"@"$DC" -k --use-ldaps --dc-ip "$IP" --no-pass  --web-host 0.0.0.0 --web-port 4443
```

### **Get ACLs for a Specific Group**
```sh
# Get ACLs for a specific group
Get-DomainObjectAcl -Identity "Recruitment Managers" -Properties DisplayName, SecurityIdentifier

# Alternative syntax
Get-ObjectAcl -SamAccountName "Recruitment Managers" -ResolveGUIDs
```