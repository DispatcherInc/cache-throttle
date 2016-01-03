[![Circle CI](https://circleci.com/gh/DispatcherInc/cache-throttle.svg?style=svg&circle-token=21468a65559575349852a5bf2fbad530bb56ede2)](https://circleci.com/gh/DispatcherInc/cache-throttle)

# cache-throttle
Provides caching and throttling of promises.

- **throttler.throttleAsync**(func) - Limits the number of concurrent calls. Any additional calls will wait.
- **lockableCache.callAsync**(key, func) - Ensures a function isn't called concurrently. Any subsequent calls with same key before the first has resolved will wait and receive the same response.

Patching:
- **cachify**(func) - Returns the function wrapped with caching
- **cachifyAll**(target) - Patches all the target's methods with caching
- **throttlify**(func) - Retruns the function wrapped with throttling
- **throttlifyAll**(target) - Pathes all the target's methods with throttling

Similar definitions to bluebird's promisify:
- cachify and throttlify resemble [bluebird's promisify](http://bluebirdjs.com/docs/api/promise.promisify.html)
- cachifyAll and throttlifyAll resemble [bluebird's promisifyAll](http://bluebirdjs.com/docs/api/promise.promisifyall.html)

## Examples
```
npm install @dispatcher/cache-throttle
```
```javascript
var cacheThrottle = require('@dispatcher/cache-throttle');
var Promise = require('bluebird');
var superagent = require('superagent');
var agent = require('superagent-promise')(superagent, Promise);

var API = {
	getUsersAsync: function() { 
		return agent.get('/users/').end();
	},
	getDriversAsync: function() {
		return agent.get('/drivers/').end();
	},
	getDriverAsync: function(driverId) {
		return agent.get('/drivers/' + driverId).end();
	}
};

cacheThrottle.throttlifyAll(API, /* optional */ {
	concurrency: 1,
	queueLimit: 100,
	suffix: 'Throttled' // or leave empty to override methods
});
cacheThrottle.cachifyAll(API, /* optional */ {
	suffix: 'Cached' // or leave empty to override methods
});
// NOTE: throttling should be applied before caching
```
Or for single functions:
```javascript
var getDriversAsyncThrottled = cacheThrottle.throttlify(API.getDriversAsync, /* optional */ {context: API});
var getDriversAsyncCached = cacheThrottle.cachify(API.getDriversAsync, /* optional */  {context: API});
```
Or use `LockableCache` and `Throttler` directly:
```javascript
var throttler = new cacheThrottle.Throttler(/* optional */ {
	concurrency: 1,
	queueLimit: 100
});
var lockableCache = new cacheThrottle.LockableCache();

lockableCache.callAsync('users', function() {
	return throttler.throttleAsync(function() {
		return agent.get('/users/').end();
	});
});
```
