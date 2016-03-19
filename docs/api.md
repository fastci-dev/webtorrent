# WebTorrent API

## `WebTorrent.WEBRTC_SUPPORT`

Is WebRTC natively supported in the environment?

```js
if (WebTorrent.WEBRTC_SUPPORT) {
  // WebRTC is supported
} else {
  // Use a fallback
}
```

## `client = new WebTorrent([opts])`

Create a new `WebTorrent` instance.

If `opts` is specified, then the default options (shown below) will be overridden.

```js
{
  dht: Boolean|Object,   // Enable DHT (default=true), or options object for DHT
  maxConns: Number,      // Max number of connections per torrent (default=55)
  nodeId: String|Buffer, // DHT protocol node ID (default=randomly generated)
  peerId: String|Buffer, // Wire protocol peer ID (default=randomly generated)
  rtcConfig: Object,     // RTCPeerConnection configuration object (default=STUN only)
  tracker: Boolean,      // Whether or not to enable trackers (default=true)
  wrtc: Object           // Custom webrtc implementation (in node, specify the [wrtc](https://www.npmjs.com/package/wrtc) or [electron-webrtc](https://github.com/mappum/electron-webrtc) package)
}
```

## `client.add(torrentId, [opts], [function ontorrent (torrent) {}])`

Start downloading a new torrent. Aliased as `client.download`.

`torrentId` can be one of:

- magnet uri (string)
- torrent file (buffer)
- info hash (hex string or buffer)
- parsed torrent (from [parse-torrent](https://github.com/feross/parse-torrent))
- http/https url to a torrent file (string)
- filesystem path to a torrent file (string)

If `opts` is specified, then the default options (shown below) will be overridden.

```js
{
  announce: [],              // Torrent trackers to use (added to list in .torrent or magnet uri)
  getAnnounceOpts: Function, // Custom callback to allow sending extra parameters to the tracker
  maxWebConns: Number,       // Max number of simultaneous connections per web seed
  path: String,              // Folder to download files to (default=`/tmp/webtorrent/`)
  store: Function            // Custom chunk store (must follow [abstract-chunk-store](https://www.npmjs.com/package/abstract-chunk-store) API)
}
```

If `ontorrent` is specified, then it will be called when **this** torrent is ready to be
used (i.e. metadata is available). Note: this is distinct from the 'torrent' event which
will fire for **all** torrents.

If you want access to the torrent object immediately in order to listen to events as the
metadata is fetched from the network, then use the return value of `client.add`. If you
just want the file data, then use `ontorrent` or the 'torrent' event.

## `client.seed(input, [opts], [function onseed (torrent) {}])`

Start seeding a new torrent.

`input` can be any of the following:

- path to the file or folder on filesystem (string) (Node.js only)
- W3C [File](https://developer.mozilla.org/en-US/docs/Web/API/File) object (from an `<input>` or drag and drop)
- W3C [FileList](https://developer.mozilla.org/en-US/docs/Web/API/FileList) object (basically an array of `File` objects)
- Node [Buffer](https://nodejs.org/api/buffer.html) object (works in [the browser](https://www.npmjs.com/package/buffer))
- Node [Readable stream](https://nodejs.org/api/stream.html#stream_class_stream_readable) object

Or, an **array of `string`, `File`, `Buffer`, or `stream.Readable` objects**.

If `opts` is specified, it should contain the following types of options:

- options for [create-torrent](https://github.com/feross/create-torrent#createtorrentinput-opts-function-callback-err-torrent-) (to allow configuration of the .torrent file that is created)
- options for `client.add` (see above)

If `onseed` is specified, it will be called when the client has begun seeding the file.

**Note:** Every torrent is required to have a name. If one is not explicitly provided
through `opts.name`, one will be determined automatically using the following logic:

- If all files share a common path prefix, that will be used. For example, if all file
  paths start with `/imgs/` the torrent name will be `imgs`.
- Otherwise, the first file that has a name will determine the torrent name. For example,
  if the first file is `/foo/bar/baz.txt`, the torrent name will be `baz.txt`.
- If no files have names (say that all files are Buffer or Stream objects), then a name
  like "Unnamed Torrent <id>" will be generated.

**Note:** Every file is required to have a name. For filesystem paths or W3C File objects,
the name is included in the object. For Buffer or Readable stream types, a `name` property
can be set on the object, like this:

```js
var buf = new Buffer('Some file content')
buf.name = 'Some file name'
client.seed(buf, cb)
```

## `client.on('torrent', function (torrent) {})`

Emitted when a torrent is ready to be used (i.e. metadata is available and store is
ready). See the torrent section for more info on what methods a `torrent` has.

## `client.remove(torrentId, [function callback (err) {}])`

Remove a torrent from the client. Destroy all connections to peers and delete all saved
file data. If `callback` is specified, it will be called when file data is removed.

## `client.destroy([function callback (err) {}])`

Destroy the client, including all torrents and connections to peers. If `callback` is specified, it will be called when the client has gracefully closed.

## `client.torrents[...]`

An array of all torrents in the client.

## `client.get(torrentId)`

Returns the torrent with the given `torrentId`. Convenience method. Easier than searching
through the `client.torrents` array. Returns `null` if no matching torrent found.

## `client.downloadSpeed`

Total download speed for all torrents, in bytes/sec.

## `client.uploadSpeed`

Total upload speed for all torrents, in bytes/sec.

## `client.progress`

Total download progress for all **active** torrents, from 0 to 1.

## `client.ratio`

Aggregate "seed ratio" for all torrents (uploaded / downloaded), from 0 to 1.


# Torrent API

## `torrent.infoHash`

Get the info hash of the torrent.

## `torrent.magnetURI`

Get the magnet URI of the torrent.

## `torrent.files[...]`

An array of all files in the torrent. See the file section for more info on what methods
the file has.

## `torrent.swarm`

The attached [bittorrent-swarm](https://github.com/feross/bittorrent-swarm) instance.

## `torrent.received`

Get total bytes received from peers (including invalid data).

## `torrent.downloaded`

Get total bytes received from peers (excluding invalid data).

## `torrent.timeRemaining`

Get the time remaining in millis if downloading.

## `torrent.downloadSpeed`

Torrent download speed, in bytes/sec.

## `torrent.uploadSpeed`

Torrent upload speed, in bytes/sec.

## `torrent.progress`

Torrent download progress, from 0 to 1.

## `torrent.ratio`

Torrent "seed ratio" (uploaded / downloaded), from 0 to 1.

## `torrent.path`

Get the torrent download location.

## `torrent.destroy()`

Alias for `client.remove(torrent)`.

## `torrent.addPeer(addr)`

Adds a peer to the underlying [bittorrent-swarm](https://github.com/feross/bittorrent-swarm) instance.

Returns `true` if peer was added, `false` if peer was blocked by the loaded blocklist.

## `torrent.addWebSeed(url)`

Adds a web seed to the [bittorrent-swarm](https://github.com/feross/bittorrent-swarm) instance.

## `torrent.select(start, end, [priority], [notify])`

Selects a range of pieces to prioritize starting with `start` and ending with `end` (both inclusive)
at the given `priority`. `notify` is an optional callback to be called when the selection is updated
with new data.

## `torrent.deselect(start, end, priority)`

Deprioritizes a range of previously selected pieces.

## `torrent.critical(start, end)`

Marks a range of pieces as critical priority to be downloaded ASAP. From `start` to `end`
(both inclusive).

## `torrent.createServer([opts])`

Create an http server to serve the contents of this torrent, dynamically fetching the
needed torrent pieces to satisfy http requests. Range requests are supported.

Returns an `http.Server` instance (got from calling `http.createServer`). If `opts` is specified, it is passed to the `http.createServer` function.

Visiting the root of the server `/` will show a list of links to individual files. Access
individual files at `/<index>` where `<index>` is the index in the `torrent.files` array
(e.g. `/0`, `/1`, etc.)

Here is a usage example:

```js
var client = new WebTorrent()
var magnetURI = 'magnet: ...'

client.add(magnetURI, function (torrent) {
  // create HTTP server for this torrent
  var server = torrent.createServer()
  server.listen(port) // start the server listening to a port

  // visit http://localhost:<port>/ to see a list of files

  // access individual files at http://localhost:<port>/<index> where index is the index
  // in the torrent.files array

  // later, cleanup...
  server.close()
  client.destroy()
})
```

## `torrent.pause()`

Temporarily stop connecting to new peers. Note that this does not pause new incoming
connections, nor does it pause the streams of existing connections or their wires.

## `torrent.resume()`

Resume connecting to new peers.

## `torrent.on('done', function () {})`

Emitted when all the torrent's files have been downloaded

Here is a usage example:

```js
torrent.on('done', function(){
  console.log('torrent finished downloading');
  torrent.files.forEach(function(file){
     // do something with file
  })
})
```

## `torrent.on('download', function (chunkSize) {})`

Emitted every time a new chunk of data arrives, it's useful for reporting the current torrent status, for instance:

```js
torrent.on('download', function(chunkSize){
  console.log('chunk size: ' + chunkSize);
  console.log('total downloaded: ' + torrent.downloaded);
  console.log('download speed: ' + torrent.downloadSpeed);
  console.log('progress: ' + torrent.progress);
  console.log('======');
})
```

## `torrent.on('wire', function (wire) {})`

Emitted whenever a new peer is connected for this torrent. `wire` is an instance of
[`bittorrent-protocol`](https://github.com/feross/bittorrent-protocol), which is a
node.js-style duplex stream to the remote peer. This event can be used to specify
[custom BitTorrent protocol extensions](https://github.com/feross/bittorrent-protocol#extension-api).

Here is a usage example:

```js
var MyExtension = require('./my-extension')

torrent1.on('wire', function (wire, addr) {
  console.log('connected to peer with address ' + addr)
  wire.use(MyExtension)
})
```

See the `bittorrent-protocol`
[extension api docs](https://github.com/feross/bittorrent-protocol#extension-api) for more
information on how to define a protocol extension.

# File API

## `file.name`

File name, as specified by the torrent. *Example: 'some-filename.txt'*

## `file.path`

File path, as specified by the torrent. *Example: 'some-folder/some-filename.txt'*

## `file.length`

File length (in bytes), as specified by the torrent. *Example: 12345*

## `file.select()`

Selects the file to be downloaded, but at a lower priority than files with streams.
Useful if you know you need the file at a later stage.

## `file.deselect()`

Deselects the file, which means it won't be downloaded unless someone creates a stream
for it.

## `stream = file.createReadStream([opts])`

Create a [readable stream](https://nodejs.org/api/stream.html#stream_class_stream_readable)
to the file. Pieces needed by the stream will be prioritized highly and fetched from the
swarm first.

You can pass `opts` to stream only a slice of a file.

```js
{
  start: startByte,
  end: endByte
}
```

Both `start` and `end` are inclusive.

## `file.getBuffer(function callback (err, buffer) {})`

Get the file contents as a `Buffer`.

The file will be fetched from the network with highest priority, and `callback` will be
called once the file is ready. `callback` must be specified, and will be called with a an
`Error` (or `null`) and the file contents as a `Buffer`.

```js
file.getBuffer(function (err, buffer) {
  if (err) throw err
  console.log(buffer) // <Buffer 00 98 00 01 01 00 00 00 50 ae 07 04 01 00 00 00 0a 00 00 00 00 00 00 00 78 ae 07 04 01 00 00 00 05 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ...>
})
```

## `file.appendTo(rootElem, [function callback (err, elem) {}])`

Show the file in a the browser by appending it to the DOM. This is a powerful function
that handles many file types like video (.mp4, .webm, .m4v, etc.), audio (.m4a, .mp3,
.wav, etc.), images (.jpg, .gif, .png, etc.), and other file formats (.pdf, .md, .txt,
etc.).

The file will be fetched from the network with highest priority and streamed into the
page (if it's video or audio). In some cases, video or audio files will not be streamable
because they're not in a format that the browser can stream so the file will be fully downloaded before being played. For other non-streamable file types like images and PDFs,
the file will be downloaded then displayed.

`rootElem` is a container element (CSS selector or reference to DOM node) that the content
will be shown in. A new DOM node will be created for the content and appended to
`rootElem`.

`callback` will be called once the file is visible to the user. `callback` is called
with an `Error` (or `null`) and the new DOM node that is displaying the content.

```js
file.appendTo('#containerElement', function (err, elem) {
  if (err) throw err // file failed to download or display in the DOM
  console.log('New DOM node with the content', elem)
})
```

## `file.renderTo(elem, [function callback (err, elem) {}])`

Like `file.appendTo` but renders directly into given element (or CSS selector).

## `file.getBlobURL(function callback (err, url) {})`

Get a url which can be used in the browser to refer to the file.

The file will be fetched from the network with highest priority, and `callback` will be
called once the file is ready. `callback` must be specified, and will be called with a an
`Error` (or `null`) and the Blob URL (`String`).

```js
file.getBlobURL(function (err, url) {
  if (err) throw err
  var a = document.createElement('a')
  a.download = file.name
  a.href = url
  a.textContent = 'Download ' + file.name
  document.body.appendChild(a)
})
```