# [GoAccess](http://goaccess.io/)
> an open source real-time web log analyzer and interactive viewer that runs in a terminal in *nix systems


## Install

### Mac
```sh
brew install goaccess
```

## Example for nginx `access.log`
```sh
`which goaccess` -f /var/log/nginx/access.log -a -g -H -M --real-os --log-format='%h %^[%d:%t %^] "%r" %s %b "%R" "%u"' --date-format="%d/%b/%Y" --time-format='%H:%M:%S' > report.html
```