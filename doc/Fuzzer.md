# Fuzzer

Once we have built the `mediasoup-worker-fuzzer` target in a Linux environment with `fuzzer` capable `clang` (see `make fuzzer` documentation in [Building](Building.md)) we can then execute the fuzzer binary at `worker/out/Release/mediasoup-worker-fuzzer`.

*NOTE:* From now on, we assume we are in the mediasoup `worker` directory.


## Related documentation

* [libFuzzer documentation](http://llvm.org/docs/LibFuzzer.html)
* [libFuzzer Tutorial](https://github.com/google/fuzzer-test-suite/blob/master/tutorial/libFuzzerTutorial.md)
* [webrtcH4cKS ~ Lets get better at fuzzing in 2019](https://webrtchacks.com/lets-get-better-at-fuzzing-in-2019-heres-how/)
* [OSS-Fuzz](https://github.com/google/oss-fuzz) - Continuous fuzzing of open source software ("fuzz for me")


## Corpus files

The `fuzzer/corpora` directory has corpus directories taken from Chromium projects. They should be use to feed fuzzer with appropriate input.

However, given how `libFuzzer` works, the first directory given as command line parameter is not just used for reading corpus files, but also to store newly generated ones. So, it's recommended to pass `fuzzer/new-corpus` as first directory. Such a directory is gitignored (for now).


## Crash reports

When the fuzzer detects an issue it generates a crash report file which contains the bytes given as input. Those files can be individually used later (instead of passing corpus directories) to the fuzzer to verify that the issue has been fixed.

The `fuzzer/reports` directory should be used to store those new crash reports. In order to use it, the following option must be given to the fuzzer executable:

```bash
-artifact_prefix=fuzzer/reports/
```

## Others

It's recommended to (also) pass the following options to the fuzzer:

* `-max_len=1400`: We don't need much more input size.

For memory leak detection enable the following environment variable:

* `LSAN_OPTIONS=verbosity=1:log_threads=1`

The `fuzzer/fuzzer.cpp` file reads some custom environment variables to decide which kind of checks perform:

* `MS_FUZZ_STUN=1`: Do STUN checks.
* `MS_FUZZ_RTCP=1`: Do RTCP checks.
* `MS_FUZZ_RTP=1`: Do RTP checks.
* If none of them is given, then **all** checks are enabled.


## Usage examples

* Detect memory leaks and just check STUN:

```bash
$ MS_FUZZ_STUN=1 LSAN_OPTIONS=verbosity=1:log_threads=1 ./out/Release/mediasoup-worker-fuzzer -artifact_prefix=fuzzer/reports/ -max_len=1400 fuzzer/new-corpus fuzzer/corpora/stun-corpus
```

* Detect memory leaks and just check RTCP:

```bash
$ MS_FUZZ_RTCP=1 LSAN_OPTIONS=verbosity=1:log_threads=1 ./out/Release/mediasoup-worker-fuzzer -artifact_prefix=fuzzer/reports/ -max_len=1400 fuzzer/new-corpus fuzzer/corpora/rtcp-corpus
```

* Detect memory leaks and check everything:

```bash
$ LSAN_OPTIONS=verbosity=1:log_threads=1 ./out/Release/mediasoup-worker-fuzzer -artifact_prefix=fuzzer/reports/ -max_len=1400 fuzzer/new-corpus fuzzer/corpora/stun-corpus fuzzer/corpora/rtcp-corpus fuzzer/corpora/rtp-corpus
```

* Verify that a specific crash is fixed:

```bash
$ LSAN_OPTIONS=verbosity=1:log_threads=1 ./out/Release/mediasoup-worker-fuzzer fuzzer/reports/crash-f39771f7a03c0e7e539d4e52f48f7adad8976404
```