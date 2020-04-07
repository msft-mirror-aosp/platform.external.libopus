# Fuzzer for libopus decoder

## Plugin Design Considerations
The fuzzer plugin for opus decoder is designed based on the understanding of the
codec and tries to achieve the following:

##### Maximize code coverage

This fuzzer provides support for both single stream and multi stream inputs,
thus enabling fuzzing for API's provided for single stream as well as multi
stream.

Following arguments are passed to OPUS_DEC_CREATE_API:

1. Sampling frequency (parameter name: `Fs`)
2. Number of channels (parameter name: `channels`)

| Parameter| Valid Values| Configured Value|
|------------- |-------------| ----- |
| `Fs` | `8000 ` `12000 ` `16000 ` `24000 ` `48000 ` | Derived from Byte-9 of input stream|
| `channels`   | `1 ` `2 ` | Derived from Byte-9 of input stream |

##### Maximize utilization of input data
The plugin feeds the entire input data to the codec. Frame sizes are determined only
after the call to extractor, so in absence of call to extractor,
we feed the entire data to the decoder.
This ensures that the plugin tolerates any kind of input (empty, huge,
malformed, etc) and doesnt `exit()` on any input and thereby increasing the
chance of identifying vulnerabilities.

## Build

This describes steps to build opus_dec_fuzzer and opus_multistream_dec_fuzzer binary.

## Android

### Steps to build
Build the fuzzer
```
  $ mm -j$(nproc) opus_dec_fuzzer
  $ mm -j$(nproc) opus_multistream_dec_fuzzer
```

### Steps to run
Create a directory CORPUS_DIR and copy some opus files to that folder.
Push this directory to device.

To run on device
```
  $ adb sync data
  $ adb shell /data/fuzz/arm64/opus_dec_fuzzer/opus_dec_fuzzer CORPUS_DIR
  $ adb shell /data/fuzz/arm64/opus_multistream_dec_fuzzer/opus_multistream_dec_fuzzer CORPUS_DIR
```
To run on host
```
  $ $ANDROID_HOST_OUT/fuzz/x86_64/opus_dec_fuzzer/opus_dec_fuzzer CORPUS_DIR
  $ $ANDROID_HOST_OUT/fuzz/x86_64/opus_multistream_dec_fuzzer/opus_multistream_dec_fuzzer CORPUS_DIR
```

## References:
 * http://llvm.org/docs/LibFuzzer.html
 * https://github.com/google/oss-fuzz
