```sh
tar -xvf file.tar

tar -cvf archive_name.tar folder_name/ #compress
tar -czvf archive_name.tar.gz folder_name #compress and zip it.

tar -cJvf zipThis.tar.xz zipThis/ #create and extract archives “.tar”
tar -xzvf archive.tar.gz -C directory
tar -xjvf archive.tar.bz2
```

| Option | Description                    |
| ------ | ------------------------------ |
| -x     | extract                        |
| -cvf   | c: create, f: file, v: Verbose |
| -cJvf  | best compression               |
| xxd    | deal with hex files.           |
|        |                                |
