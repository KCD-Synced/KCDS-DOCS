# Messages between resources

With `kcds.core.Messages`, one resource can call another by name, either waiting for a reply or
simply notifying it. It suits a dependency that may be absent, because neither side needs the other's
functions. Both resources must run on the same side. For server-to-client traffic, use `Events`.

## Receiving

Each message type has one handler in a resource, and the latest `handle` call for a type wins.

```ts
kcds.core.Messages.handle('post', (payload, reply) => {
  const { text } = payload as { text: string };
  reply({ ok: Boolean(text) });
});
```

A handler should always answer, even when it fails. Otherwise the caller waits with no result, so put
`reply` in a `try`/`finally` when the work can throw.

## Asking and waiting

To get an answer, call `Messages.request(resource, type, payload?)` and await it:

```ts
const answer = (await kcds.core.Messages.request('blog', 'post', { text: 'hello' })) as {
  ok: boolean;
};
if (answer.ok) {
  console.info('posted');
}
```

| What happens                    | The promise                                                 |
| ------------------------------- | ----------------------------------------------------------- |
| The handler calls `reply(value)` | resolves a tick later, and only the first reply counts      |
| The resource has no handlers    | rejects with `Resource is not available`                    |
| No handler for this type        | rejects with `Resource does not handle this message type`   |
| The handler throws first        | rejects with that error                                     |
| The handler never replies       | keeps waiting, and rejects only when either resource stops  |

There is no built-in time limit. If a silent handler must not hold you up, race the request against
a timer:

```ts
function withTimeout<T>(request: Promise<T>, ms: number): Promise<T> {
  return new Promise<T>((resolve, reject) => {
    const timer = setTimeout(() => reject(new Error('timed out')), ms);
    request.then(
      (value) => {
        clearTimeout(timer);
        resolve(value);
      },
      (problem) => {
        clearTimeout(timer);
        reject(problem);
      },
    );
  });
}

await withTimeout(kcds.core.Messages.request('blog', 'post', { text: 'hello' }), 2000);
```

## Notifying without a reply

`Messages.send(resource, type, payload?)` does not wait for anything. `reply` does nothing in this
case, and a missing resource or handler causes no error.

```ts
kcds.core.Messages.send('audit', 'login', { player: 'Henry' });
```

## Restarts

When a resource stops or restarts, its handlers go, and requests waiting on it reject.
