# api documentation for  [line-reader (v0.4.0)](https://github.com/nickewing/line-reader#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-line-reader.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-line-reader) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-line-reader.svg)](https://travis-ci.org/npmdoc/node-npmdoc-line-reader)
#### Asynchronous, buffered, line-by-line file/stream reader

[![NPM](https://nodei.co/npm/line-reader.png?downloads=true)](https://www.npmjs.com/package/line-reader)

[![apidoc](https://npmdoc.github.io/node-npmdoc-line-reader/build/screenCapture.buildNpmdoc.browser.%252Fhome%252Ftravis%252Fbuild%252Fnpmdoc%252Fnode-npmdoc-line-reader%252Ftmp%252Fbuild%252Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-line-reader/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-line-reader/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-line-reader/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Nick Ewing",
        "email": "nick@nickewing.net"
    },
    "bugs": {
        "url": "https://github.com/nickewing/line-reader/issues"
    },
    "dependencies": {},
    "description": "Asynchronous, buffered, line-by-line file/stream reader",
    "devDependencies": {
        "mocha": "^2.4.5"
    },
    "directories": {
        "lib": "./lib"
    },
    "dist": {
        "shasum": "17e44818da0ac335675ba300954f94ef670e66fd",
        "tarball": "https://registry.npmjs.org/line-reader/-/line-reader-0.4.0.tgz"
    },
    "gitHead": "bd38cc8c5483e4b6799c01bc9b88819fda1461c7",
    "homepage": "https://github.com/nickewing/line-reader#readme",
    "keywords": [
        "file",
        "line",
        "reader",
        "scanner"
    ],
    "license": "MIT",
    "main": "./lib/line_reader",
    "maintainers": [
        {
            "name": "nickewing",
            "email": "nick@nickewing.net"
        }
    ],
    "name": "line-reader",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://nickewing@github.com/nickewing/line-reader.git"
    },
    "scripts": {
        "test": "mocha test/line_reader.js"
    },
    "url": "https://github.com/nickewing/line-reader",
    "version": "0.4.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module line-reader](#apidoc.module.line-reader)
1.  [function <span class="apidocSignatureSpan">line-reader.</span>eachLine (filename, options, iteratee, cb)](#apidoc.element.line-reader.eachLine)
1.  [function <span class="apidocSignatureSpan">line-reader.</span>open (filenameOrStream, options, cb)](#apidoc.element.line-reader.open)



# <a name="apidoc.module.line-reader"></a>[module line-reader](#apidoc.module.line-reader)

#### <a name="apidoc.element.line-reader.eachLine"></a>[function <span class="apidocSignatureSpan">line-reader.</span>eachLine (filename, options, iteratee, cb)](#apidoc.element.line-reader.eachLine)
- description and source-code
```javascript
function eachLine(filename, options, iteratee, cb) {
  if (options instanceof Function) {
    cb = iteratee;
    iteratee = options;
    options = undefined;
  }
  var asyncIteratee = iteratee.length === 3;

  var theReader;
  var getReaderCb;

  open(filename, options, function(err, reader) {
    theReader = reader;
    if (getReaderCb) {
      getReaderCb(reader);
    }

    if (err) {
      if (cb) cb(err);
      return;
    }

    function finish(err) {
      reader.close(function(err2) {
        if (cb) cb(err || err2);
      });
    }

    function newRead() {
      if (reader.hasNextLine()) {
        setImmediate(readNext);
      } else {
        finish();
      }
    }

    function continueCb(continueReading) {
      if (continueReading !== false) {
        newRead();
      } else {
        finish();
      }
    }

    function readNext() {
      reader.nextLine(function(err, line) {
        if (err) {
          finish(err);
        }

        var last = !reader.hasNextLine();

        if (asyncIteratee) {
          iteratee(line, last, continueCb);
        } else {
          if (iteratee(line, last) !== false) {
            newRead();
          } else {
            finish();
          }
        }
      });
    }

    newRead();
  });

  // this hook is only for the sake of testing; if you choose to use it,
  // please don't file any issues (unless you can also reproduce them without
  // using this).
  return {
    getReader: function(cb) {
      if (theReader) {
        cb(theReader);
      } else {
        getReaderCb = cb;
      }
    }
  };
}
```
- example usage
```shell
...
The 'eachLine' function reads each line of the given file.  Upon each new line,
the given callback function is called with two parameters: the line read and a
boolean value specifying whether the line read was the last line of the file.
If the callback returns 'false', reading will stop and the file will be closed.

var lineReader = require('line-reader');

lineReader.eachLine('file.txt', function(line, last) {
  console.log(line);

  if (/* done */) {
    return false; // stop reading
  }
});
...
```

#### <a name="apidoc.element.line-reader.open"></a>[function <span class="apidocSignatureSpan">line-reader.</span>open (filenameOrStream, options, cb)](#apidoc.element.line-reader.open)
- description and source-code
```javascript
function open(filenameOrStream, options, cb) {
  if (options instanceof Function) {
    cb = options;
    options = undefined;
  }

  var readStream;

  if (typeof filenameOrStream.read == 'function') {
    readStream = filenameOrStream;
  } else if (typeof filenameOrStream === 'string' || filenameOrStream instanceof String) {
    readStream = fs.createReadStream(filenameOrStream);
  } else {
    cb(new Error('Invalid file argument for LineReader.open.  Must be filename or stream.'));
    return;
  }

  readStream.pause();
  createLineReader(readStream, options, cb);
}
```
- example usage
```shell
...
  console.log("I'm done!!");
});

For more granular control, 'open', 'hasNextLine', and 'nextLine' maybe be used
to iterate a file (but you must 'close' it yourself):

// or read line by line:
lineReader.open('file.txt', function(err, reader) {
  if (err) throw err;
  if (reader.hasNextLine()) {
    reader.nextLine(function(err, line) {
      try {
        if (err) throw err;
        console.log(line);
      } finally {
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
