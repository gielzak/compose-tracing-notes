# DAC instructions

Capture a trace in Android Studio: https://goo.gle/compose-tracing

# Manual capture

Capturing a trace without Android Studio.

Note: the same constraints (e.g. API >= 30) apply as in the DAC instructions.

## Example app

git clone https://github.com/android/nowinandroid.git

## Add dependencies

1. Find latest on https://maven.google.com

2. Add `compose-tracing` and force the latest `tracing-perfetto` as per below. Make sure to _NOT_ release the app with `tracing-perfetto-binary` as it’s huge.
```
implementation("androidx.compose.runtime:runtime-tracing:1.0.0-alpha01")
implementation("androidx.tracing:tracing-perfetto:1.0.0-alpha06")
implementation("androidx.tracing:tracing-perfetto-binary:1.0.0-alpha06")
```

3. Verify through `./gradlew app:dependencies`

## Generate a record command

1. Generate a record command using: https://ui.perfetto.dev/#!/record/instructions
2. Manually add the `track_event` data source section as per example below.

Example recording command:
```
adb shell perfetto \
  -c - --txt \
  -o /data/misc/perfetto-traces/trace \
<<EOF

buffers: {
    size_kb: 63488
    fill_policy: RING_BUFFER
}
buffers: {
    size_kb: 2048
    fill_policy: RING_BUFFER
}
data_sources: {
    config {
        name: "track_event"
    }
}
duration_ms: 10000
flush_period_ms: 30000
incremental_state_config {
    clear_period_ms: 5000
}

EOF
```

# Capture a trace

1. Launch the app and prepare section you want to trace

2. Enable tracing in the app by issuing a broadcast (expect `exitCode: 1`, more exit codes in [source](https://cs.android.com/android/platform/frameworks/support/+/androidx-main:tracing/tracing-perfetto-common/src/main/java/androidx/tracing/perfetto/PerfettoHandshake.kt;l=216;drc=125437f5e8b829997e6e4244274c5dc52194e33f))

```
# set app package variable, e.g. com.google.samples.apps.nowinandroid.debug
# can be found through `adb shell ps -ef` or `adb shell cmd package list packages`
package=<your app process>

# issue a broadcast to enable tracing
adb shell am broadcast \
    -a androidx.tracing.perfetto.action.ENABLE_TRACING \
    $package/androidx.tracing.perfetto.TracingReceiver    
```

3. Start your recording command mentioned previously.

## Open the trace

1. Pull (`adb pull <location>`) the trace from the device (location specified in the record command).

2. Open in https://ui.perfetto.dev
