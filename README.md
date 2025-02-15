# otp_agent

This library provides a way for Gleam programmers to create stateful and OTP
compatible server process that can respond to both synchronous and
asynchronous messages. It intends to be a common building block for Gleam OTP
applications, similar to `gen_server` for Erlang and `GenServer` for Elixir.

Being OTP compatible agent processes support OTP tracing, error reporting, and
will fit into a supervision tree.


## Example

Imagine we wanted to implement an agent that works like a stack, allowing us
to push and pop elements.

```rust
import gleam/otp/agent.{Continue, Reply}

pub fn start_link(initial stack) {
  agent.start_link(fn() { agent.Ready(stack) })
}

pub fn push(onto pid, push item) {
  let push_fn = fn(stack) {
    Continue([item | stack])
  }
  agent.async(pid, push_fn)
}

pub fn pop(from pid) {
  let pop_fn = fn(stack) {
    case stack {
      [] ->
        Reply(with: Error(Nil), then: Continue([]))

      [elem | rest] ->
        Reply(with: Ok(elem), then: Continue(rest))
    }
  }
  agent.sync(pid, pop_fn)
}
```

It can then be used like so:

```rust
import gleam/expect

pub fn stack_agent_test() {
  let Ok(agent) = start_link(initial: ["Hello"])

  // Popping returns the first element
  expect.equal(
    pop(from: agent),
    Ok("Hello"),
  )

  // Popping a second time returns nothing as the stack is empty
  expect.equal(
    pop(from: agent),
    Error(Nil),
  )

  push(onto: agent, item: "World!")

  expect.equal(
    pop(from: agent),
    Ok("World!"),
  )
}
```
