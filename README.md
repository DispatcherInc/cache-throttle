[![Circle CI](https://circleci.com/gh/DispatcherInc/cache-throttle.svg?style=svg&circle-token=21468a65559575349852a5bf2fbad530bb56ede2)](https://circleci.com/gh/DispatcherInc/cache-throttle)

# cache-throttle
Provides function caching and throttling.

- **throttler.throttleAsync**(func) - Limits the number of concurrent calls. Any additional calls will wait.
- **lockableCache.callAsync**(key, func) - Ensures a function isn't called concurrently. Any subsequent calls with same key before the first has resolved will wait and receive the same response.

Patching:
- **cachify**(obj, method) - Patches the object's method with caching
- **cachifyAll**(obj) - Patches all the object's methods with caching
- **throttlify**(obj, method) - Patches the object's method with throttling
- **throttlifyAll**(obj) - Pathes all the object's methods with caching

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

cacheThrottle.throttlifyAll(API, {
	concurrency: 1,
	queueLimit: 100
});
cacheThrottle.cachifyAll(API);
// NOTE: throttling should be applied before caching
```
Or for single functions:
```javascript
cacheThrottle.throttlify(API, 'getDriversAsync');
cacheThrottle.cachify(API, 'getDriversAsync');
```
Or use `LockableCache` and `Throttler` directly:
```javascript
var throttler = new cacheThrottle.Throttler();
var lockableCache = new cacheThrottle.LockableCache();

lockableCache.callAsync('users', function() {
	return throttler.throttleAsync(function() {
		return agent.get('/users/').end();
	});
});
```
