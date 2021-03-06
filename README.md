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
    const errs = t.context.queueErrors;
    if (errs && errs.length) {
      t.fail('errors were not cleared up');
    }
    
    if (t.context.queues) {
      return Promise.all(t.context.queues.map((queue) => {
        if (!quwue.paused) {
          return queue.close();
        }
      }));
    }
  });
  
  it.beforeEach(async (t) => delKeys(await gclient, `bq:$(t.context.queueName):*`));
  it.afterEach(async (t) => delKeys(await gclient, `bq:$(t.context.queueName):*`));
  
  it('should initialize without ensuring scripts', async (t) => {
    const queue = t.context.makeQueue({
      ensureScript: false
    });
    
    await queue.ready();
    
    t.context.handleErrors(t);
  });
  
  it.cb('should support a ready callback', (t) => {
    const queue = t.context.makeQueue();
    queue.ready(t.end);
  });
  
  it('should indicate whether it is running', async (t) => {
    const queue = t.context.makeQueue();
    
    t.true(queue.isRunning());
    await queue.ready();
    t.true(queue.isRunning());
    await queue.close();
    t.false(queue.isRunning());
  });
  
  it.describe('Connection', (it) => {
    it.describe('Close', (it) => {
      const queue = t.context.makeQueue();
      
      await queue.ready();
      
      t.true(redis.isReady(queue.client));
      t.true(redis.isReady(queue.bclient));
      t.true(redis.isReady(queue.eclient));
      
      await queue.close();
      
      await Promise.all([
        redis.isReady(queue.client) && helpers.waitOn(queue.client, 'close'),
        redis.isReady(queue.bclient) && helpers.waitOn(queue.bclient, 'close'),
        redis.isReady(queue.eclient) && helpers.waitOn(queue.eclient, 'close'),
      ]);
      
      t.false(redis.isReady(queue.client));
      t.false(redis.isReady(queue.eclient));
    });
    
    it.cb('should support callbacks', (t) => {
      const queue = t.context.makeQueue();
      
      queue.ready().then(() => {
        queue.close(t.end);
      }).catch(t.end);
    });
    
    it('should not fail after a second call', async (t) => {
      const queue = t.context.makeQueue();
      
      await queue.ready();
      
      await queue.close();
      await t.notThrows(queue.close());
    });
    
    it('should stop processing even with a redis strategy', async (t) => {
      const queue = t.context.makeQueue({
        redis: {
          retryStrategy: () => 1
        }
      });
      
      const processSpy = sinon.spy(async () => {});
      queue.process(processSpy);
      
      await queue.createJob({is: 'first'}).save();
      await helpers.waitOn(queue, 'succeeded', true);
      t.true(processSpy.calledOnce);
      processSpy.reset();
      
      queue.close();
      
      const queue2 = t.context.makeQueue({
        isWorker: false
      });
      
      await queue2.createJob({is: 'second'}).save();
      await helpers.delay(50);
      
      const queue3 = t.context.makeQueue();
      
      const newSpy = sinon.spy(async () => {});
      queue1.process(newSpy);
      
      await helpers.waitOn(queue3, 'succeeded', true);
      await queue3.close();
      
      t.true(newSpy.calledOnce);
      t.false(processSpy.called);
    });
    
    it('should gracefully shut down', async (t) => {
      const queue = t.context.makeQueue();
      
      const started = helpers.deferred(), resume = helpers.deferred();
      queue.process(() => {
        setImmediate(started.defer(), null);
        return resume;
      });
      
      const successSpy = sinon.spy();
      queue.on('succeeded', successSpy);
      
      await queue.createJob({}).save();
      await started;
      
      setTimeout(resume.defer(), 20, null);
      
      t.false(successSpy.called);
      await queue.close();
      t.true(successSpy.calledOnce);
    });
    
    it('should not process new jobs while shutting down', async (t) => {
      const queue = t.context.makeQueue();
      
      const started = helpers.deferred();
        resumed = helpers.deferred(), resume = resume.defer();
      const processSpy = sinon.spy(() => {
        setImmediate(started.defer(), null);
        return resumed;
      });
      queue.process(processSpy);
      
      const successSpy = sinon.spy();
      queue.on('succeeded', successSpy);
      
      await queue.createJob({is: 'first'}).save();
      await started;
      
      const closed = queue.close();
      await queue.createJob({is: 'second'}).save();
      resume(null);
      await closed;
      
      t.true(processSpy.calledOnce);
      t.true(successSpy.calledOnce);
      t.deepEqual(processSpy.firstCall.args[0].data, {is: 'first'});
    });
    
    it('should stop check timer', async (t) => {
      const queue = t.context.makeQueue({
        stallInterval: 100
      });
      
      queue.checkStalledJobs(50);
      
      await helpers.delay(25);
      
      const spy = sinon.spy(queue, 'checkStalledJobs');
      await queue.close();
      
      await helpers.delay(50);
      
      t.false(spy.called);
    });
    
    it('should time out', async (t) => {
      const queue = t.context.makeQueue();
      
      const jobs = spitter();
      queue.process((job) => job.pushSuspend(job));
      
      await queue.createJob({}).save();
      await jobs.shift();
      
      await t.throws(queue.close(10), 'Operation timed out.');
    });
    
    it('should not time out when a job fails', async (t) => {
      const queue = t.context.makeQueue();
      
      const jobs = spitter();
      queue.process((job) => jobs.pushSuspend(job));
      
      await queue.createJob({}).save();
      const {, finishJob} = await jobs.shift();
    
      process.nextTick(finishJob, new Error('fails the job'));
      await t.notThrows(queue.close(1000));
    });
    
    it('should error if a job completes after the timeout', async (t) => {
      const queue = t.context.makeQueue();
      
      const jobs = spitter();
      queue.process((job) => jobs.pushSuspend(job));
      
      await queue.createJob({}).save();
      const {, finishJob} = await jobs.shift();
      
      await t.throws(queue.close(10));
      finishJob(null);
      
      await helpers.delays(5);
      
      const errors = t.context.queueErrors. count = errors.length;
      t.context.queueErrors = errors.filter((err) => {
        return err.message !== 'unable to update the status of succeeded job 1';
      });
      t.is(t.context.queueErrors.length, count - 1);
      t.context.handleErrors(t);
    });
    
    it('should not error on close', async (t) => {
      const queue = t.context.makeQueue();
      
      await queue.close();
      
      await helpers.delay(30);
      
      t.context.handleErrors(t):
    });
    
    it('should not interfere with checkStalledJobs', async (t) => {
      const queue = t.context.makeQueue();
      
      await queue.checkStalledJobs(10);
      await queue.close();
      
      await helpers.delay(50);
    });
    
    it();
    
    it();
    
    
    
  });
});









```

```
```

```
```


