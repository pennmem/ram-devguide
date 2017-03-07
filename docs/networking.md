# Network messages

The task and host computers communicate via JSON messages over a [ZeroMQ][]
`PAIR` socket. Messages have the following format:

```json
{
  "type": "MESSAGETYPE",
  "data": <>,
  "aux": <>,
  "time": 1234.5678
}
```

where `type` is the type of message, `data` contains parameters or metadata
related to the specific type of message, `aux` may be used for secondary data,
and `time` is a timestamp in milliseconds since the epoch. Only `type` is
strictly required, though generally speaking, messages should include a
timestamp and explicitly include `data` and `auxdata` as `null`.

Message types fall generally into three categories:

1. To-host messages: Messages that go from the task laptop to the host PC.
   These mostly consist of messages indicating changes in the state of the
   experiment but also include other information needed by the host.
2. From-host messages: Messages that go from the host PC to the task laptop.
   These are mostly commands that indicate, for example, to start, pause,
   resume, or stop an experiment.
3. Bidirectional messages: Messages that can go in either direction. These
   include data request-reply messages, heartbeat messages to ensure the
   connection is still alive, and others.

In the following, message types are described and expected values for `data`
and `aux` are listed (assumed `null` when omitted).

[ZeroMQ]: http://zeromq.org/

## To-host message types

### `EXPNAME`

Transmits current experiment name to ensure no mismatch.

```json
{
  "data": "name of experiment e.g., FR1"
}
```

### `VERSION`

Transmits task version number.

```json
{
  "data": "version number in format x.y.z"
}
```

###  `SESSION`

Transmits session information.

```json
{
  "data": {"session_number": <int>}
}
```

###  `SUBJECTID`

Transmits the subject's ID.

```json
{
  "data": "subject ID"
}
```

### `DEFINE`

Not used? Sends list of possible states in the experiment.

```json
{
  "data": ["<list of states>"]
}
```

### `READY`

Indicates that the task laptop is awaiting a `START` message.

### `ALIGNCLOCK`

Request a clock synchronization procedure (not used in System 3 yet).

### `TRIAL`

Send the current trial/list number with a session.

```json
{
  "data": {"trial": <int>}
}
```

### `WORD`

Transmits word or words being displayed.

```json
{
  "data": "RHINO"
}
```
**TODO**: make it: `data: {"words": [<string>, ...]}`.

### `MATH`

Transmits data related to math distractor periods.

```json
{
  "data": {
    "problem": "1 + 1 + 7 = ",
    "response": "100",
    "response_time_ms": 1828,
    "correct": false
  }
}
```

### `STATE`

Indicates a change in the state of the experiment. `STATE` types are listed in
more detail below.

```json
{
  "data": {
    "name": "STATETYPE",
    "value": true,
    "meta": {}
  }
}
```

The `meta` object can store whatever else and is meant to be backwards- and
forwards-compatible. For PS4/x experiments, `meta` looks like:

```json
{
  "phase_type": "<string>"
}
```

where `phase_type` is one of `BASELINE`, `PS`, `STIM`, or `NON-STIM`.

#### State messages

The term "state" here refers to any part of the experiment in which a definite
beginning and end exists (these can be overlapping). Possible states are:

* `PRACTICE` - a practice session
* `NON-STIM ENCODING` - deprecated
* `RETRIEVAL` - retrieval portion of a verbal task session
* `DISTRACT` - distraction (math) portion of a task
* `INSTRUCT` - instruction period
* `COUNTDOWN` - break between active periods with countdown
* `WAITING` - waiting for the session to start
* `WORD` - word display in verbal tasks
* `ORIENT` - orientation character display (between active/inactive periods)
* `MIC TEST` - microphone testing
* `VOCALIZATION` - voice activity detected

## From-host message types

### `SYNCED`

Clock synchronization complete.

### `START`

Ready to start the experiment.

## Bidirectional message types

### `HEARTBEAT`

Used to ensure that each end is still responsive.

```json
{
  "data": <integer, interval in ms that heartbeat is sent>
}
```

### `CONNECTED`

Handshake message when the connection is established

### `SYNC`

Used for clock synchronization/network latency testing.

```json
{
  "aux": <integer>
}
```

The `aux` field keeps track of the order of sent `SYNC` messages (indicated in
the `num` field from the host).

### `WORDPOOL`

Request or receive a pool of words for a session. From the task laptop to the
host PC:

```json
{
  "data": {
    "session": 0
  }
}
```

From the host PC to the task laptop:

```json
{
  "data": {
    "session": 0,
    "lists": [
      [<list 0>], [<list 1>], ..., [<list n-1>]
    ],
    "lures": null
  }
}
```

Logic for word pool creation is delegated to the host PC, thus it is only valid
to send this message after the experiment type is known by the host. A list of
lures is sent only for experiment types requiring them, otherwise the `lures`
list is length 0 or `null`.

## Message sequence

1. Task started, waits for `CONNECTED`
2. Host sends `CONNECTED`
3. Task sends `EXPNAME`, `VERSION`, `SESSION`, `SUBJECTID`
4. If required, task sends `WORDPOOL`, waits for `WORDPOOL` response
5. Task sends `READY`, waits for `START`
6. Host sends `START`
7. Task starts, sending `STATE` and other messages as appropriate
8. Science happens

## Notes for future changes

* All `data` parameters should in the nuture be `null` or an object/dict.
* Unused messages will be deprecated and eliminated.
* `EXPNAME`, `VERSION`, `SESSION`, and `SUBJECTID` should be combined. Possibly
  also `WORDPOOL`.
* The `aux` field should be removed; all data can be carried in the `data`
  field.
* Deprecated state types will be removed.
* Vocalization states may incorporate voice recognition in the future for some
  tasks.
* `SYNC` messages should be modified to be consistent between host and task.
* For sharing between host and task computers, it would be nice if there is a
  common library that defines these message types.
