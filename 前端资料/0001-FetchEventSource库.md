# FetchEventSource库

库的地址
https://github.com/Azure/fetch-event-source

## Usage

```javascript
// BEFORE:
const sse = new EventSource('/api/sse');
sse.onmessage = (ev) => {
    console.log(ev.data);
};

// AFTER:
import {fetchEventSource} from '@microsoft/fetch-event-source';

await fetchEventSource('/api/sse', {
    onmessage(ev) {
        console.log(ev.data);
    }
});
```

You can pass in all the other parameters exposed by the default fetch API, for example:

```javascript
const ctrl = new AbortController();
fetchEventSource('/api/sse', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        foo: 'bar'
    }),
    signal: ctrl.signal,
});
```

You can add better error handling, for example:

```javascript
class RetriableError extends Error {
}

class FatalError extends Error {
}

fetchEventSource('/api/sse', {
    async onopen(response) {
        if (response.ok && response.headers.get('content-type') === EventStreamContentType) {
            return; // everything's good
        } else if (response.status >= 400 && response.status < 500 && response.status !== 429) {
            // client-side errors are usually non-retriable:
            throw new FatalError();
        } else {
            throw new RetriableError();
        }
    },
    onmessage(msg) {
        // if the server emits an error message, throw an exception
        // so it gets handled by the onerror callback below:
        if (msg.event === 'FatalError') {
            throw new FatalError(msg.data);
        }
    },
    onclose() {
        // if the server closes the connection unexpectedly, retry:
        throw new RetriableError();
    },
    onerror(err) {
        if (err instanceof FatalError) {
            throw err; // rethrow to stop the operation
        } else {
            // do nothing to automatically retry. You can also
            // return a specific retry interval here.
        }
    }
});
```
