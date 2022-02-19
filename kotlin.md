### Flow
Kotlin `flow` has built in backpressure, in case parallel processing is required to process a `flow`, kotlin `channel` can be used to provide backpressure, i.e. fan-out pattern.

```
val channel = produce(capacity = concurrency) { flow.collect { send(it) } }
repeat(concurrency) { launch(Dispatchers.IO) { for (message in channel) { /* process */ } } }
```
