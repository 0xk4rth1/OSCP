
Adding export path:

`export PATH=/tmp:$PATH`

it will append `/tmp` in the first order, so whenever the binary execution occurs it'll lookout for the binary from first to last, so it will look in the attacker controlled /tmp folder. So, we can easily hijacking any binary without the full path (/usr/bin/curl). if a program or script uses 'curl' then it will execute /tmp/curl

