### For c# apps:
```
dotnet publish -c Release -r win-x64 --self-contained false -p:PublishSingleFile=true -p:PublishTrimmed=true
```

### For c/c++ apps:
```
x86_64-w64-mingw32-gcc Printconfig.c -o --shared -o Printconfig.dll
```