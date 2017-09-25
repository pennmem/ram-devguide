# Network messages

The task and host computers communicate via JSON messages over a [ZeroMQ][]
`PAIR` socket on TCP port 8889. Messages have the following format:

```json
{
  "type": "MESSAGETYPE",
  "data": <>,
  "time": 1234.5678
}
```

where `type` is the type of message, `data` contains parameters or metadata
related to the specific type of message, and `time` is a timestamp in 
milliseconds since the epoch. Only `type` is strictly required, though 
generally speaking, messages should at least include a timestamp as well.

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
are listed (assumed `null` when omitted).

[ZeroMQ]: http://zeromq.org/

## To-host message types

###  `SESSION`

Transmits session information.

```json
{
  "data": {
    "name": "PS4_FR5",
    "version": "5.0.0",
    "subject": "subject ID",
    "session_number": 0
  }
}
```

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
    ...
  }
}
```

Other data is dependent on the particular state. For example, for `WORD` states,
this would include `phase_type` which can be one of `BASELINE`, `PS`, `STIM`, or
`NON-STIM`.

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

### `CONNECTED`

Handshake message when the connection is established

### `STIM`

Sent whenever the host PC applies stimulation pulses.

```json
{
  "data": {
    "EXPNAME": "experiment name",
    "stim_electrode_pair": "string of labels",
    "stim_frequency": <frequency in mHz>,
    "stim_amplitude": <amplitude in uA>,
    "stim_duration": <duration in ms>
  }
}
```

### `SYNC`

Used for clock synchronization/network latency testing.

```json
{
  "aux": <integer>
}
```

The `aux` field keeps track of the order of sent `SYNC` messages (indicated in
the `num` field from the host).

## Message sequence

1. Task waits for `CONNECTED` then sends it back to the host
2. Task sends `SESSION`, waits for `START` from host
3. Host sends `START`
4. Task sends `TRIAL`
5. Task starts, sending `STATE` and other messages as appropriate
6. Science happens

Note that once the host has received `CONNECTED`, it will process *any* message
sent by the task laptop. It is **imperative** that the task laptop wait for a
`START` message before the experiment is allowed to proceed.

## Notes for future changes

* Deprecated state types will be removed.
* Vocalization states may incorporate voice recognition in the future for some
  tasks.
* `SYNC` messages should be modified to be consistent between host and task.
* For sharing between host and task computers, it would be nice if there is a
  common library that defines these message types.

## Helpful notes

During development of the tasks, crashes can sometimes leave zombie processes
alive that prevent re-binding the socket. On Mac, the PID can be found with the
following command:

```
lsof -i tcp:8889
```
