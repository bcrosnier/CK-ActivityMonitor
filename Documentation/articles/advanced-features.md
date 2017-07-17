# Advanced features

Some features can help when trying to improve output or performance issues.

## Custom outputs on one ActivityMonitor

You can add your own output handler for a single `ActivityMonitor` by creating an `IActivityMonitorClient` and registering it by calling `monitor.Output.RegisterClient(client)`.

The following example adds an `ActivityMonitorConsoleClient` to _one_ monitor:

```csharp
public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    IActivityMonitorClient client = new ActivityMonitorConsoleClient();
    m.Output.RegisterClient(client);
}
```

## Apply a custom configuration to _all_ new ActivityMonitors

You can set up some code that will be executed _every time an ActivityMonitor is created_ using the static `ActivityMonitor.AutoConfiguration` event.

The following example adds an `ActivityMonitorConsoleClient` to _every new monitor_ after it:

```csharp
public void Startup() {
    ActivityMonitor.AutoConfiguration += (monitor) =>
    {
        IActivityMonitorClient client = new ActivityMonitorConsoleClient();
        monitor.Output.RegisterClient(client);
    };
}
```

## Filtering ActivityMonitor logs

`ActivityMonitor` instances obey a `LogFilter` to decide which logs to process and output. This filter can be set by the monitors' custom outputs; the `GrandOutput` does not set it..

You can set the default `LogFilter` on the static property `ActivityMonitor.LogFilter`.

It can be overridden on every monitor, either permanently by setting its `MinimalFilter` property, or temporarily by calling `TemporarilySetMinimalFilter()`, which returns an `IDisposable`.

A `LogFilter` is made of two `LogLevel`s: one for log _lines_, one for log _groups_.

Groups that do not pass the filter are still opened: lines inside these groups are processed and outputted as if the group was never opened.

All log levels are processed by default.

## Lambda log messages

A log message can be replaced by a lambda that returns it; if the log does not pass the filter, this lambda will not be executed.
This can be handy when using logs to perform IO calls like database queries.

```csharp
public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    m.MinimalFilter = LogFilter.Off;
    m.Error(() => "My error message");
}
```

## Automatic tags

An `ActivityMonitor` can be configured to have tags sent automatically, without having to add them to every log call.

```csharp
public static CKTrait LogTag = ActivityMonitor.Tags.Register("MY_LOG_TAG");
public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    m.AutoTags = LogTag;
    m.Info("Log message with tags");
}
```

You can temporarily set automatic tags by calling `TemporarilySetAutoTags()`, which returns an `IDisposable`.
