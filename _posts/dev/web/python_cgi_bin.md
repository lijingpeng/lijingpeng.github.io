title: Python CGI-BIN
date: 2015-11-18 13:19:56
tags: python
category: web
---

```python
import BaseHTTPServer
import CGIHTTPServer
import cgitb; cgitb.enable()  ## This line enables CGI error reporting

server = BaseHTTPServer.HTTPServer
handler = CGIHTTPServer.CGIHTTPRequestHandler
server_address = ("0.0.0.0", 8082)
handler.cgi_directories = ["/cgi-bin"]

httpd = server(server_address, handler)
httpd.serve_forever()
```

CGI-BIN下的py文件要有执行权限。

```python
#!/usr/bin/env python

import cgi
import urllib
import json

form = cgi.FieldStorage()

query_str = form.getvalue('query')
query = "http://0.0.0.0:9999/query?query=%s" % query_str

exec_query = urllib.urlopen(query)
print "Status: 200 OK"
print "Content-Type: application/json"
print ""
print exec_query.read()
```

HTML:
```html
<html>
    <head>
        <meta http-equiv="content-type" content="text/html; charset=utf-8">

        <title>test</title>
        <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
        <script>

            $(function()
            {
                $('#clickme').click(function(){
                    alert('Im going to start processing');

                    $.ajax({
                        url: "/scripts/ajaxpost.py",
                        type: "post",
                        datatype:"json",
                        data: {'key':'value','key2':'value2'},
                        success: function(response){
                            alert(response.message);
                            alert(response.keys);
                        }
                    });
                });
            });

        </script>
    </head>
    <body>
        <button id="clickme"> click me </button>
    </body>

</html>
```

参考： 
- https://pointlessprogramming.wordpress.com/2011/02/13/python-cgi-tutorial-2/
- http://www.runoob.com/python/python-cgi.html
- http://stackoverflow.com/questions/10721244/ajax-posting-to-python-cgi