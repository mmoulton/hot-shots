# hot-shots

A Node.js client for [Etsy](http://etsy.com)'s [StatsD](https://github.com/etsy/statsd) server, Datadog's [DogStatsD](http://docs.datadoghq.com/guides/dogstatsd/) server, and [InfluxDB's](http://influxdb.com) [Telegraf](https://github.com/influxdb/telegraf) StatsD server.

This project is a fork off of [node-statsd](https://github.com/sivy/node-statsd).  This project includes all changes in node-statsd, all open PRs to node-statsd when possible, and some additional goodies (like Telegraf support).

[![Build Status](https://secure.travis-ci.org/brightcove/hot-shots.png?branch=master)](http://travis-ci.org/brightcove/hot-shots)

## Installation

```
$ npm install hot-shots
```

## Usage

All initialization parameters are optional.

Parameters (specified as an options hash):
* `host`:        The host to send stats to `default: localhost`
* `port`:        The port to send stats to `default: 8125`
* `prefix`:      What to prefix each stat name with `default: ''`
* `suffix`:      What to suffix each stat name with `default: ''`
* `globalize`:   Expose this StatsD instance globally? `default: false`
* `cacheDns`:    Cache the initial dns lookup to *host* `default: false`
* `mock`:        Create a mock StatsD instance, sending no stats to the server? `default: false`
* `globalTags`:  Tags that will be added to every metric `default: []`
* `maxBufferSize`: If larger than 0,  metrics will be buffered and only sent when the string length is greater than the size. `default: 0`
* `bufferFlushInterval`: If buffering is in use, this is the time in ms to always flush any buffered metrics. `default: 1000`
* `telegraf`:    Use Telegraf's StatsD line protocol, which is slightly different than the rest `default: false`

All StatsD methods other than event have the same API:
* `name`:       Stat name `required`
* `value`:      Stat value `required except in increment/decrement where it defaults to 1/-1 respectively`
* `sampleRate`: Sends only a sample of data to StatsD `default: 1`
* `tags`:       The Array of tags to add to metrics `default: []`
* `callback`:   The callback to execute once the metric has been sent or buffered

If an array is specified as the `name` parameter each item in that array will be sent along with the specified value.

The event method has the following API:

* `title`:       Event title `required`
* `text`:        Event description `default is title`
* `options`:     Options for the event
  * `date_happened`    Assign a timestamp to the event `default is now`
  * `hostname`         Assign a hostname to the event.
  * `aggregation_key`  Assign an aggregation key to the event, to group it with some others.
  * `priority`         Can be ‘normal’ or ‘low’ `default: normal`
  * `source_type_name` Assign a source type to the event.
  * `alert_type`       Can be ‘error’, ‘warning’, ‘info’ or ‘success’ `default: info`
* `tags`:       The Array of tags to add to metrics `default: []`
* `callback`:   The callback to execute once the metric has been sent

```javascript
  var StatsD = require('hot-shots'),
      client = new StatsD();

  // Timing: sends a timing command with the specified milliseconds
  client.timing('response_time', 42);

  // Increment: Increments a stat by a value (default is 1)
  client.increment('my_counter');

  // Decrement: Decrements a stat by a value (default is -1)
  client.decrement('my_counter');

  // Histogram: send data for histogram stat (DataDog and Telegraf only)
  client.histogram('my_histogram', 42);

  // Gauge: Gauge a stat by a specified amount
  client.gauge('my_gauge', 123.45);

  // Set: Counts unique occurrences of a stat (alias of unique)
  client.set('my_unique', 'foobar');
  client.unique('my_unique', 'foobarbaz');

  // Event: sends the titled event (DataDog only)
  client.event('my_title', 'description');

  // Incrementing multiple items
  client.increment(['these', 'are', 'different', 'stats']);

  // Sampling, this will sample 25% of the time the StatsD Daemon will compensate for sampling
  client.increment('my_counter', 1, 0.25);

  // Tags, this will add user-defined tags to the data (DataDog and Telegraf only)
  client.histogram('my_histogram', 42, ['foo', 'bar']);

  // Using the callback
  client.set(['foo', 'bar'], 42, function(error, bytes){
    //this only gets called once after all messages have been sent
    if(error){
      console.error('Oh noes! There was an error:', error);
    } else {
      console.log('Successfully sent', bytes, 'bytes');
    }
  });

  // Sampling, tags and callback are optional and could be used in any combination (DataDog and Telegraf only)
  client.histogram('my_histogram', 42, 0.25); // 25% Sample Rate
  client.histogram('my_histogram', 42, ['tag']); // User-defined tag
  client.histogram('my_histogram', 42, next); // Callback
  client.histogram('my_histogram', 42, 0.25, ['tag']);
  client.histogram('my_histogram', 42, 0.25, next);
  client.histogram('my_histogram', 42, ['tag'], next);
  client.histogram('my_histogram', 42, 0.25, ['tag'], next);
```

## DogStatsD and Telegraf functionality

Some of the functionality mentioned above is specific to DogStatsD or Telegraf.  They will not do anything if you are using the regular statsd client.
* globalTags parameter- DogStatsD or Telegraf
* tags parameter- DogStatsD or Telegraf
* telegraf parameter- Telegraf
* histogram method- DogStatsD or Telegraf
* event method- DogStatsD

## Errors

In the event that there is a socket error, `hot-shots` will allow this error to bubble up.  If you would like to catch the errors, just attach a listener to the socket property on the instance.

```javascript
client.socket.on('error', function(error) {
  return console.error("Error in socket: ", error);
});
```

If you want to catch errors in sending a message then use the callback provided.

## Submitting changes

Thanks for considering making any updates to this project!  Here are the steps to take in your fork:

1. Make sure you've added any new tests needed
2. Run "npm install && npm test"
3. Update the HEAD section in CHANGES.md with a description of what you have done
4. Push your changes and create the PR, and we'll try to get this merged in right away

## Name

Why is this project named hot-shots?  Because:

1. It's impossible to find another statsd name on npm
2. It's the name of a dumb movie
3. No good reason

## License

hot-shots is licensed under the MIT license.
