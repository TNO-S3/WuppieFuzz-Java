# WuppieFuzz

TNO developed WuppieFuzz, a coverage-guided REST API fuzzer developed on top of
LibAFL, targeting a wide audience of end-users, with a strong focus on
ease-of-use, explainability of the discovered flaws and modularity. WuppieFuzz
supports all three settings of testing (black box, grey box and white box).

## WuppieFuzz-Java

For Java there is an existing coverage agent (Jacoco) which is
[available online](https://www.jacoco.org/jacoco/trunk/index.html). It can be
used like this:

```bash
java -javaagent:/jacoco/jacocoagent.jar=output=tcpserver <...more arguments...>
```

`jacocoagent.jar` can be downloaded [here](https://www.jacoco.org/jacoco/) as
part of the `.zip`-release. The file is found in the `lib` directory.

### Protocol and format

Jacoco's communication protocol and format are strongly coupled, since message
sizes are dynamically determined from the coverage information. Communication
works as follows:

1. Receive a request for coverage. The structure of this request should be as
   follows:

   1. header byte
   2. array of command-type bytes; only the byte for the dump-command is used by
      us
   3. array of bytes requesting coverage dumps (or not)?
   4. array of bytes requesting resets (or not).

   The three lists all have only one byte in our case.

2. Respond on the TCP-stream with a series of "blocks", each of which is
   composed of bytes in order:

   1. a byte indicating the block type
   2. depending on the block type:
      1. header - format version information (currently there only seems to be
         one format and one version, at least that we support)
      2. session info - info about when the Jacoco agent started and last dumped
         coverage
      3. **execution data** - it seems there should be **one such block for each
         class that has at least some coverage**. It contains just three pieces
         of info:
         1. class id: u64
         2. class name: String (variable length)
         3. boolean array of whether probes were hit or not (supplies its own
            size)
      4. no contents, for the final block.
