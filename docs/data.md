# System 3.0

## Task laptop data files

System 3.0 stores much of the data on the task laptop in the form of plaintext
files. The directory structure is something like:

```
data
└── <experiment>
    └── <subject>
        ├── RAM_wordpool.txt
        ├── RAM_wordpool_noAcc.txt
        ├── config.py
        ├── configbackup
        ├── experiment.log
        ├── messages.log
        ├── sconfig.py
        ├── sconfigbackup
        ├── session_0
        ├── session_1
        ├── session_10
        ├── session_11
        ├── session_12
        ├── session_13
        ├── session_14
        ├── session_15
        ├── session_16
        ├── session_17
        ├── session_2
        ├── session_3
        ├── session_4
        ├── session_5
        ├── session_6
        ├── session_7
        ├── session_8
        ├── session_9
        └── state
```

and within session directories:

```
0.lst
1.lst
10.lst
11.lst
12.lst
13.lst
14.lst
15.lst
16.lst
17.lst
18.lst
19.lst
2.lst
20.lst
21.lst
22.lst
23.lst
24.lst
3.lst
4.lst
5.lst
6.lst
7.lst
8.lst
9.lst
audio.sndlog
keyboard.keylog
math.log
p.lst
p.wav
session.log
video.vidlog
```

Notes on the stored files:

- Log files (`*.log`, `*.vidlog`, `*.keylog`, `*.sndlog`) are plaintext,
  tab-delimited files.
- The entire wordpool used is stored in `RAM_wordpool.txt`
  (`RAM_wordpool_noAcc.txt` is the same but with some unicode normalization
  applied; it does not appear to be used anymore).
- List files (`*.lst`) are the individual plaintext word lists used in each
  session. `0.lst` is the practice session.
- `*.wav` files record audio from recall portions of tasks. The naming
  convention is `p.wav` for the practice session, then numbered sessions for the
  rest.

## Host PC

Directory structure:

```
data
└── <subject>
    └── <experiment>
        ├── config_files
        │   ├── <ens config file>.bin
        │   └── <ens config file>.csv
        ├── experiment_config.json
        ├── output.log
        └── session_0
            └── <timestamped directory>
                ├── config_files
                │   ├── <ens config file>.bin
                │   └── <ens config file>.csv
                ├── eeg_timeseries.h5
                ├── event_log.json
                ├── experiment_config.json
                └── output.log
```

# System 3.1

## Task laptop

Options for improving data storage for System 3.1:

- Storing data in SQL. This would have the advantages of not needing a complex
  directory structure and possibly making events creation easier (or at least
  more standardized).
- Storing no data on the task laptop and instead sending it all to the host PC
  for storage.
    - For System 3.0, this is already (mostly) implemented on the task laptop
      side apart from transmitting audio files.
    - A potential problem with this is that we would still want to record data
      locally as a backup; dealing with uploading in two possible scenarios
      could become a challange.
