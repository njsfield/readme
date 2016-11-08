# Event Emitter & Streams

## Investigate EventEmitters and the event module (Tom & Steve)

- What are event emitters and event listeners?
- What's the observer pattern aka the publish/ subscribe pattern?

## Streams and the stream module

![Pic demonstrating duplex](./rw.png)

### What are streams?

- Streams are unix pipes that let you easily read data from a source and pipe it to a destination.
- They behave similarly to UNIX pipes.
- Streams are instances of EventEmitter.
- Where Event Emitter allows callbacks of data, streams use Buffer objects to transfer data.
- Streams can accept various types of data, including text and binary data.

### What types of streams are there?

- <b>Readable</b> (i.e. read from source, e.g. request in http).
  After initialising, a readable stream triggers a 'data' event when chunks of data are received, this event relinquishes ownership of that chunk of data. Eventually, an 'end' event is fired, which signals the end of the stream.
  ```javascript
  var fs = require('fs'); //fileSystem module
  var readableStream = fs.createReadStream('file.txt'); //create read buffer
  var data = '';

  readableStream.on('data', function(chunk) {
      data+=chunk; // add chunk of data when chunk of data is ready
  });

  readableStream.on('end', function() {
      console.log(data); //when stream has finished reading the data.
  });
  ```

- <b>Writable</b> (write to destination, e.g. response in http).
  Takes the writable chunks of data and writes it to the destination. Mostly works in unison with readable streams. Returns a boolean after operation indicating whether it was successful or not.
  ```javascript
  var fs = require('fs'); // Filesystem
  var readableStream = fs.createReadStream('file1.txt'); // Creating read stream from file1
  var writableStream = fs.createWriteStream('file2.txt'); // Creating write stream for file2
  readableStream.setEncoding('utf8'); // Ensuring buffer encoding is utf-8 (which is 256 possible characters).

  readableStream.on('data', function(chunk) {
      writableStream.write(chunk); // writing contents of file1 => file2
  });
  ```

- <b>Duplex</b> (can both read & write).
  ![Pic demonstrating duplex](./duplex.png)
  An example of a duplex stream would be a socket, which can be used to allow communication between a client and server.
  To create a custom duplex stream, create a class which inherits from the Duplex abstract class, implement the write and read methods. Duplex streams inherit from transform streams...

- <b>Transform</b> (write to, modify, and then read from).
  ![Pic demonstrating transform](./transform.png)
  Transform streams allow modification/transformation of the data flowing to and from it:
  ```javascript
  var Transform = require('stream').Transform;  // pulls in transform module
  var inherits = require('util').inherits; // deprecated. Use 'extends'

  module.exports = JSONEncode;

  function JSONEncode(options) {  
    if ( ! (this instanceof JSONEncode))
      return new JSONEncode(options);

    if (! options) options = {};
    options.objectMode = true;
    Transform.call(this, options); // transforming data from JSON data
  }

  inherits(JSONEncode, Transform);

  JSONEncode.prototype._transform = function _transform(obj, encoding, callback) {  
    try {
      obj = JSON.stringify(obj);
    } catch(err) {
      return callback(err);
    }

    this.push(obj);
    callback();
  };
  ```

As streams read from a Buffer object, you can set the encoding if need be;

```javascript
readableStream.setEncoding('utf8');
```

### What are pipes?

What's an easy way of connecting a readable stream to a writable stream? Pipes! In UNIX piping, you can do the following;

```
$~ cat file1.txt | nano file2.txt
```
This shows how you can take the outputted contents of file1.txt, and pipe it into the contents of file2.txt.
This shows that you can take the output of the cat command and then pass it as an input to the nano command.

In node, similar results can be achieved. Instead of calling the 'data' event on the readable stream and returning the result on 'end' to a writeable stream, you can simply connect the readable stream to the writable stream via the .pipe method;

```javascript
var fs = require('fs'); // Magic
var readableStream = fs.createReadStream('file1.txt'); // Super Magic
var writableStream = fs.createWriteStream('file2.txt'); // Super Magic

readableStream.pipe(writableStream); //Wizardry
```

### Command Line Examples

- The following example shows how to list files in the current directory (ls), retain only the lines of ls output containing the string "key" (grep), and view the result in a scrolling page (less);
```
$~ ls -l | grep key | less
```
- The following example shows how to inspect all system processes then retain only the processes that contain 'chrome'.
```
$~ ps aux | grep chrome
```


## (All)

These concepts can get quite abstract. What are some common examples of EventEmitters and Streams in node we're likely to come across as a group? Can you think of any small program ideas or code samples you can use to demonstrate and talk through these concepts with the group?
