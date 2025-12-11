## Upload file
### Uploading a file via HTTP/HTTPS (PUT method, for replacing or creating resources):
```sh
curl -X PUT -T "/path/to/yourfile.txt" "http://example.com/new/resource/file"
```


### Uploading a file via HTTP/HTTPS (POST method, common for web forms):
```sh
curl -X POST -F "file=@/path/to/yourfile.txt" http://example.com/api/upload
```