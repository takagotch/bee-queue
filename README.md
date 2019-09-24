### bee-queue
---
https://github.com/bee-queue/bee-queue

```js
// test/queue-test.js
import {describe} from 'ava-spec';

import Queue from '../lib/queue';

function reef(n = 1) {
  const done = helpers.deferred(), end = done.defer();
  return {
    done,
    next() {
      --n;
      if (n < 0) return false;
      if (n === 0) end();
      return true;
    }
  };
}

async function recordUntil(emitter, tracker, trackedEvnets, lastEvent) {
  const recordedEvents = [];
  
  const done = helpers.waitOn(emitter, lastEvent);
  for (let event of trackedEvents) {
    recordEvents.push([evnet, ...values]);
    };
    emitter.on(event, handler);
    done.then(() => emitter.removeListener(evnet, handler));
  }
  
  await done;
  return recordedEvents;  
}




```

```
```

```
```


