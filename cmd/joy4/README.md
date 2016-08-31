# JOY4 Media Server

`joy4` command line is a powerful media server with features: 

- HTTP RESTful API  and Callback
- Support RTMP/HTTP-FLV


- Multi-channel streaming: 1-publisher-N-subscriber
- Set buffer time/size/GOP-count
- Stream proxy
- Change publisher while streaming

# Documentation 

Stream stats:

- `connecting`

- `checkingParams` Stream is wait checking params result (RTMP publish/play command)
- `connected`
- `subscribing` Stream is a subscriber in channel
- `publishing` Stream is a publisher in channel

# Install

```bash
go install github.com/nareix/cmd/joy4
joy4 -httpapi=:8080 -http=:8081 -rtmp=:1935
```

try `ffmpeg -re -i input.flv -f flv rtmp://localhost:1935/1`

try `ffplay rtmp://localhost:1935/1` and `ffplay http://localhost:8081/1.flv`

# Config

Enable HTTP Callback

```json
joy4 -httpcallback=http://localhost:8082/callback
```

RTMP check params in callback

- will trigger `/event/streamCheckParams` callback when RTMP connection accepted

```
joy4 -rtmpcheckparams
```

# HTTP API

Get stream info

```
GET /stat/streams
{
  "1": {
    "createdAt":"2006-01-02T15:04:05Z07:00", 
    "elapsed": 42.222, // in second
    "type":"rtmp",  // rtmp/http-flv/hls-mem
    "stat":"subscribing",
    "rtmp":{
      "command":"publish",
      "connectParams":{
        "tcUrl":"rtmp://8.8.8.8/xxoo",
        ....
      },
    },
    "remoteAddr":"192.168.1.1:14242",
    "audio":{"type":"AAC", "samplerate":44100, "channelCount":2},
    "video":{"type":"H264", "width":1024, "height":768},
    "bitrate":15000,
    "time":42.003, // current position in second
    "bytes":12345,  // network transfered bytes
  },
  ...
}
  
GET /stat/stream/1
```

Get channel info

```
GET /stat/channels
{
  "channel-1": {
    "createdAt":"2006-01-02T15:04:05Z07:00",
    "elapsed":42.222,
    "subscriberCount":233,
    "bufferSize":102400,
    "bufferUsed":102400,
    "publisher":"1", // current publisher
  },
  ...
}
  
GET /stat/channel/1
```

Create channel

```
POST /channel/create
{
  "id":"channel-1",
  "bufferSize":1000000, // optional
  "bufferDuration":"120s", // optional
  "bufferGopCount": 4, // optional
}

200 {}
400 {"err":"ChannelExists"}
```

Set Channel Publisher

```
POST /channel/setPublisher
{
  "id":"channel-1",
  "streamId":"1",
}

200 {}
400 {"err":"StreamNotExists/ChannelExists/ChannelAlreadyHasPublisher"}
```

Add Channel Subscriber

```
POST /channel/addSubscriber
{
  "id":"channel-1",
  "streamId":"1",
  "cursor"
}

200 {}
400 {"err":"StreamNotExists/ChannelExists"}
```

Close Stream

```
POST /stream/close 
{
  "id":"1",
}

200 {}
400 {"err":"StreamNotExists"}
```

Close Channel

```
POST /channel/close
{
  "id":"channel-1",
}

200 {}
400 {"err":"ChannelNotExists"}
```

New stream

```
POST /stream/new
{
  "id":"1", // optional
  "url":"rtmp://...",
}

200 {"id":""}
```

Accept/Reject stream when stream in `checkingParams` state

```
POST /stream/{accept,reject}
{
  "id":"1",
}

200 {}
400 {"err":"StreamNotExists/StreamStateInvalid"}
```

Change stream id

```
POST /stream/changeid
{
  "id":"1",
  "newid":"2",
}

200 {}
400 {"err":"StreamNotExists/StreamExists"}
```

# HTTP Callback 

Stream check params event

```
POST /event/streamCheckParams
{
  "id":"1",
  "type":"rtmp",
  "rtmp":{
    "command":"publish",
    "connectParams":{
      "tcUrl":"rtmp://8.8.8.8/xxoo",
      ....
    },
  },
}
```

Stream connected event

```
POST /event/streamConnected
{
  "id":"xxoo",
  "type":"rtmp",
  ...
}
```

Stream Close event

```
POST /event/streamClose
{
  "id":"1",
}
```

Channel Close event

```
POST /event/channelClose
{
  "id":"channel-1",
}
```

