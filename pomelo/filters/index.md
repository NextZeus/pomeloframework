# pomelo filters
## serial , time , timeout, toobusy

### toobusy

```
//if the process is toobusy, just skip the new request
var conLogger = require('pomelo-logger').getLogger('con-log', __filename);
var toobusy = null;
var DEFAULT_MAXLAG = 70;//每秒钟接受70个请求，超出的请求，直接返回500


module.exports = function(maxLag) {
  return new Filter(maxLag || DEFAULT_MAXLAG);
};

var Filter = function(maxLag) {
  try {
    toobusy = require('toobusy');
  } catch(e) {
  }
  if(!!toobusy) {
    toobusy.maxLag(maxLag);
  }
};

Filter.prototype.before = function(msg, session, next) {
  if (!!toobusy && toobusy()) {
    conLogger.warn('[toobusy] reject request msg: ' + msg);
    var err = new Error('Server toobusy!');
    err.code = 500;
    next(err);
  } else {
    next();
  }
};

```