theme: Inter, 1
footer: @shelbyspees at #SLOconf2022
slide-dividers: #, ##

# [fit] Intro to Tracing-Based SLOs

Shelby Spees
Site Reliability Engineer
Equinix

[.hide-footer]
^ hi I’m Shelby
I’m an SRE at Equinix
and this is an introduction to tracing-based SLOs

# why tracing?

^ (2min this section)

^ tracing is great because it can capture so much data about the state of your service within the scope of each request.

## use OpenTelemetry

auto-instrumentation for HTTP and gRPC

- request duration
- status codes
- client calls vs. server responses

^ using HTTP or gRPC instrumentation like with OpenTelemetry means our traces automatically track things like request duration and status code
so out of the box we can already measure latency and error rate.

^ opentelemetry also automatically differentiates between client calls and server-side responses, which helps a lot when we want to look at different kinds of workloads

## traces are fancy structured logs

TODO example data

^ while tracing libraries do a lot of work to keep track of trace state, the actual data they generate is basically just structured logs that have a couple special fields to connect them.
BUT what those connections give us is the ability to see the **relationship** between events. that makes a huge difference when we're debugging.

## observability == exploration

-> broad view: what's slow?
-> zoom in: what's different about this slow traffic?
-> zoom out: is traffic with this particular attribute always slow? or did it change recently?
-> zoom in: where in the trace is this attribute being set?
-> zoom out: what's the latency on these spans for different values of this attribute?

^ we get the most benefit from tracing when using modern observability tools that accept our raw trace data and allow us to explore it interactively across lots of high-cardinality dimensions.
that ability to query on raw trace data means we don’t have to decide up front what math to do on what dimensions. instead, our observability tools do the math on the fly at query time.

# tracing-based SLOs

^ 2 min this section

^ of course we’re not redefining our SLOs on the fly, but if your tools support querying across lots of arbitrary dimensions then they can also support defining an SLI based on equally arbitrary dimensions.

^ this is a big deal. while latency and error rate are important for pretty much every service, the most important thing for your service, your key differentiator? it’s going to be unique to that service.
that means you might not be able to measure it with auto-instrumented trace data.

## instrument your code

^ so, it’s important to instrument your code.
what’s nice about tracing is that the data from custom instrumentation gets interwoven with existing auto-instrumented traces, which means it helps with debugging too!
rather than sending a bunch of disparate data points, we’re enriching our existing traces with more context.
data that can serve multiple purposes is just efficient, you know?

# defining a tracing-based SLI

- filter to what’s relevant
- define a condition for “good”

^ to define our SLI we need to do two things
different tools have different ways of defining SLIs for trace data, so for this talk I’m just using pseudocode for my SLI definitions

# overall traffic SLI: filter to what’s relevant

```javascript
// filter to what's relevant
IF(
    // root spans
    trace.parent_id == undefined
          AND
    // responding to client calls
    span.kind == "server"
)
```

^ (overall traffic SLI section: 2min)
^ querying across multiple spans in each trace is expensive, so for our SLI we’re generally going to look at individual spans that best represent the end-user experience.

^ for overall traffic to our Rails monolith we can look at server spans at the root of traces in our rails monolith

## example trace

TODO add screenshot

^ here’s an example of the kind of span we’re looking at. the root span of a trace tells us the duration for synchronous requests
those are the ones where there's a human at the other end waiting for a page to load, or an API call waiting for a response

## overall traffic SLI: filter to what’s relevant

```diff
    IF(
        trace.parent_id == undefined
              AND
        span.kind == "server"
              AND
+       user.staff != true
+             AND
+       user.bot != true
      )
```

^ we also want to make sure we’re getting an accurate representation of the customer experience, so let’s filter out requests from staff users and internal bot users.
these user fields come from custom instrumentation and they only apply where we have data about the user.
we want to be careful not to filter out spans where these user fields are undefined, so I'm specifically filtering out "true" for each one

## overall traffic SLI: define a condition for “good”

```javascript
// define a condition for good
// not 500 error and faster than 2s
// TODO
```

^ OpenTelemetry’s HTTP auto-instrumentation automatically captures the request duration and whether it was successful.

## provisioning SLI: filter to what’s relevant

```javascript
// TODO
```

^ provisioning is our bread and butter at equinix metal, so this

# provisioning SLI: filter to what’s relevant

^ we also have the ability to filter out test projects, so let’s do that.
I don’t want our on-call devs to get woken up by a QA team reproducing a bunch of errors on the other side of the world.

```javascript
// TODO
      AND
project.test != true
```

# provisioning SLI: define a condition for “good”

```javascript
// TODO
```

# you can do this!

- add OpenTelemetry auto-instrumentation
- observe and learn
- add custom instrumentation
-

^
