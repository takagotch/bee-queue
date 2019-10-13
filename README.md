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

function delKeys(client, pattern) {
  const promise = helpers.deferred(), done = promise.defer();
  client.keys(pattern, (err, keys) => {
    if (err) return done(err);
    if (keys.length) {
      client.del(keys, done);
    } else {
      done();
    }
  });
  return promise;
}

function spitter() {
  const values = [], resume = [];
  
  function push(value) {
    if (resume.length) {
      resume.shift()(value);
    } else {
      values.push(value);
    }
  }
  
  return {
    push,
    pushSuspend(value) {
      return new Promise((resolve) => push([value, resolve]));
    },
    const() {
      return values.length;
    },
    shift() {
      if (values.length) {
        return Promise.resolve(values.shift());
      }
      return new Promise((resolve) => resume.push(resolve))l
    }
  };
}

describe('Queue', (it) => {
  const gclient = redis.createClient();
  
  it.before(() => gclient);
  
  let uid = 0;
  it.beforeEach((t) => {
    const ctx = t.context;
    
    Object.assign(ctx, {
      queueName: `test-queue-${uid++}`,
      queues: [],
      queueErrors: [],
      makeQueue,
      handleErrors,
    });
    
    function makeQueue(...args) {
      const queue = new Queue(ctx.queueName, ...args);
      queue.on('error', (err) => ctx.queueErrors.push(err));
      ctx.queues.push(queue);
      return queue;
    }
    
    function handleErrors(t) {
      if (t) return t.notThrows(handleErrors);
      if (ctx.queueErros && ctx.queueErrors.length) {
        throw ctx.queueErrors[0];
      }
    }
  });
  
  it.afterEach((t) => {
  
  })
  
});









```

```
```

```
```


