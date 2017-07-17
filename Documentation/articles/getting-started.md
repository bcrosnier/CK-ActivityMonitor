# Getting started

Here are some instructions to get you set up.

## Install the NuGet packages

As of CK.ActivityMonitor 8, you only need to add [**`CK.ActivityMonitor.SimpleSender`**](https://www.nuget.org/packages/CK.ActivityMonitor.SimpleSender/) in the project where you want to call the `ActivityMonitor` **without configuring the output** (e.g. in a library used by something else that _does_ configure the output).

```posh
Install-Package CK.ActivityMonitor.SimpleSender -Pre
```

If you also want to **configure the logging output** (e.g. you are in an application host, or a website project, and not just a library), you also need [**`CK.Monitoring`**](https://www.nuget.org/packages/CK.Monitoring/).

```posh
Install-Package CK.Monitoring -Pre
```

## Configure the `ActivityMonitor` output

The `ActivityMonitor` output needs to be configured at the start of your application or website:

```csharp
using CK.Core;
using CK.Monitoring;
using CK.Monitoring.Handlers;

class Startup
{
    public Startup()
    {
        // Output all CK.Monitoring files in C:\Logs\
        // Note: The path must be absolute. Relative paths are not supported
        SystemActivityMonitor.RootLogPath = @"C:\Logs";
        // Prepare GrandOutput handlers
        GrandOutputConfiguration grandOutputConfig = new GrandOutputConfiguration();
        // TextFile handler: 10 000 lines per file, saved in C:\Logs\TextFile\
        grandOutputConfig.AddHandler( new TextFileConfiguration()
        {
            MaxCountPerFile = 10000,
            Path = "TextFile", // Relative to SystemActivityMonitor.RootLogPath
        } );
        GrandOutput.EnsureActiveDefault( grandOutputConfig );
    }
}
```

This initialization (and the `CK.Monitoring` package) is usually not necessary if your package is a simple library, but the host that uses your library will have to configure it.

The default `GrandOutput` configured here is a big sink in which all new `ActivityMonitor` instances will output.

## Create an `ActivityMonitor`

Just create a new instance of the `ActivityMonitor` class as you would for any other class, and use that.

```csharp
using CK.Core;

namespace My.App
{
    public class MyClass
    {
        public void MyMethod() {
            IActivityMonitor m = new ActivityMonitor();
            m.Info("Hello world");
        }
    }
}
```

You can add a _topic_ to a particular monitor by calling the constructor with a message:
```csharp
public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor("My topic");
}
```

You can _change_ the topic of a monitor by calling the `SetTopic(message)` method on `ActivityMonitor`.

The topic is merely a log line with a special tag, sent when constructing the monitor or changing it. More on tags below.

## Log levels

The `ActivityMonitor` has six log levels:
- `Debug`
- `Trace`
- `Info`
- `Warn`
- `Error`
- `Fatal`

Use any of the named extension methods to send a log line to the `ActivityMonitor`:

```csharp
public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    m.Debug("Debug line");
    m.Trace("Trace line");
    m.Info("Info line");
    m.Warn("Warn line");
    m.Error("Error line");
    m.Fatal("Fatal line");
}
```

## Logging exceptions

The `ActivityMonitor` will serialize any exception that you give it, including its contents and inner exceptions.

```csharp
public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    try
    {
        throw new DivideByZeroException();
    }
    catch (DivideByZeroException e)
    {
        m.Error("Something bad happened", e);
    }
}
```

## Adding tags to log lines

You can add tags (`CKTrait` objects) to log lines to categorize or filter them later on. The tag must be registered once, but can be kept:

```csharp
public static CKTrait LogTag = ActivityMonitor.Tags.Register("MY_LOG_TAG");

public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    m.Info("Line with tag", LogTag);
}
```

The same tag can be registered multiple times: they will always point to the same object.

More information about `CKTrait` can be found [directly in the source code of CK-Core](https://github.com/Invenietis/CK-Core/blob/master/CK.Core/CKTrait.cs).

### Combining tags

You can combine tags by separating them with a pipe (`|`) when registering them:

```csharp
public static CKTrait CombinedTag = ActivityMonitor.Tags.Register("TAG_ONE|TAG_TWO");

public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    m.Info("Line with TAG_ONE and TAG_TWO", CombinedTag);
}
```

## Groups

You can structure logs sections by using the the `Open*` methods, which return `IDisposable` groups:

```csharp
public void MyMethod() {
    IActivityMonitor m = new ActivityMonitor();
    using(m.OpenInfo("Info group"))
    {
        m.Info("Line structured inside group");
    }
}
```

Groups are mostly like regular log lines: they have the same log levels (`Debug`, `Trace`, `Info`, `Warn`, `Error`, `Fatal`), and you can call `Open*` with exceptions and/or tags, and put groups inside groups.

