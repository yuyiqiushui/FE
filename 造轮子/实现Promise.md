# 实现 promise

## 基础版本

```js
function MyPromise(fn) {
    let self = this;
    self.value = null;
    self.error = null;
    self.onFulfilled = null;
    self.onRejected = null;

    function resolve(value) {
        self.value = value;
        self.onFulfilled(self.value)
    }

    function reject(error) {
        self.error = error;
        self.onRejected(self.error)
    }
    fn(resolve, reject);
}
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
}
module.exports = MyPromise
```



## 支持同步任务

```js
function MyPromise(fn) {
    let self = this;
    self.value = null;
    self.error = null;
    self.onFulfilled = null;
    self.onRejected = null;

    function resolve(value) {
        setTimeout(() => {
            self.value = value;
            self.onFulfilled(self.value)
        }, 0)
    }

    function reject(error) {
        setTimeout(() => {
            self.error = error;
            self.onRejected(self.error)
        }, 0)
    }
    fn(resolve, reject);
}
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
}
module.exports = MyPromise
```



## 支持状态

```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function MyPromise(fn) {
    let self = this;
    self.value = null;
    self.error = null;
    self.status = PENDING;
    self.onFulfilled = null;
    self.onRejected = null;

    function resolve(value) {
        if (self.status === PENDING) {
            setTimeout(function() {
                self.status = FULFILLED;
                self.value = value;
                self.onFulfilled(self.value);
            }, 0)
        }
    }

    function reject(error) {
        if (self.status === PENDING) {
            setTimeout(function() {
                self.status = REJECTED;
                self.error = error;
                self.onRejected(self.error);
            }, 0)
        }
    }
    fn(resolve, reject);
}
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    if (this.status === PENDING) {
        this.onFulfilled = onFulfilled;
        this.onRejected = onRejected;
    } else if (this.status === FULFILLED) {
        onFulfilled(this.value)
    } else {
        onRejected(this.error)
    }
}
module.exports = MyPromise
```



## 支持链式操作

```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function MyPromise(fn) {
    const self = this;
    self.value = null;
    self.error = null;
    self.status = PENDING;
    self.onFulfilledCallbacks = [];
    self.onRejectedCallbacks = [];

    function resolve(value) {
        if (self.status === PENDING) {
            setTimeout(() => {
                self.status = FULFILLED;
                self.value = value;
                self.onFulfilledCallbacks.forEach((callback) => callback(self.value));
            }, 0)
        }
    }

    function reject(error) {
        if (self.status === PENDING) {
            setTimeout(function() {
                self.status = REJECTED;
                self.error = error;
                self.onRejectedCallbacks.forEach((callback) => callback(self.error));
            }, 0)
        }
    }
    fn(resolve, reject);
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
    if (this.status === PENDING) {
        this.onFulfilledCallbacks.push(onFulfilled);
        this.onRejectedCallbacks.push(onRejected);
    } else if (this.status === FULFILLED) {
        onFulfilled(this.value)
    } else {
        onRejected(this.error)
    }
    return this;
}
module.exports = MyPromise
```

## 支持串行异步任务

```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

function MyPromise(fn) {
  const self = this;
  self.value = null;
  self.error = null;
  self.status = PENDING;
  self.onFulfilledCallbacks = [];
  self.onRejectedCallbacks = [];

  function resolve(value) {
    console.log('resolve', value);
    if (self.status === PENDING) {
      setTimeout(() => {
        self.status = FULFILLED;
        self.value = value;
        self.onFulfilledCallbacks.forEach(callback => callback(self.value));
      }, 0);
    }
  }

  function reject(error) {
    if (self.status === PENDING) {
      setTimeout(function() {
        self.status = REJECTED;
        self.error = error;
        self.onRejectedCallbacks.forEach(callback => callback(self.error));
      }, 0);
    }
  }
  fn(resolve, reject);
}

function resolvePromise(bridgePromise, x, resolve, reject) {
  if (x instanceof MyPromise) {
    if (x.status === PENDING) {
      x.then(
        y => {
          resolvePromise(bridgePromise, y, resolve, reject);
        },
        error => {
          reject(error);
        }
      );
    } else {
      x.then(resolve, reject);
    }
  } else {
    resolve(x);
  }
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
  console.log('then');
  const self = this;
  let bridgePromise;
  onFulfilled =
    typeof onFulfilled === 'function' ? onFulfilled : value => value;
  onRejected =
    typeof onRejected === 'function'
      ? onRejected
      : error => {
          throw error;
        };
  if (self.status === FULFILLED) {
    return (bridgePromise = new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onFulfilled(self.value);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      }, 0);
    }));
  }
  if (self.status === REJECTED) {
    return (bridgePromise = new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onRejected(self.error);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      }, 0);
    }));
  }
  if (self.status === PENDING) {
    console.log('Pengding');
    return (bridgePromise = new MyPromise((resolve, reject) => {
      self.onFulfilledCallbacks.push(value => {
        try {
          let x = onFulfilled(value);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
      self.onRejectedCallbacks.push(error => {
        try {
          let x = onRejected(error);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
    }));
  }
};
MyPromise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected);
};

module.exports = MyPromise;

```

## 达到 promiseA+ 规范

```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function MyPromise(fn) {
    const self = this;
    self.value = null;
    self.error = null;
    self.status = PENDING;
    self.onFulfilledCallbacks = [];
    self.onRejectedCallbacks = [];

    function resolve(value) {
        if (value instanceof MyPromise) {
            return value.then(resolve, reject);
        }
        if (self.status === PENDING) {
            setTimeout(() => {
                self.status = FULFILLED;
                self.value = value;
                self.onFulfilledCallbacks.forEach((callback) => callback(self.value));
            }, 0)
        }
    }

    function reject(error) {
        if (self.status === PENDING) {
            setTimeout(function() {
                self.status = REJECTED;
                self.error = error;
                self.onRejectedCallbacks.forEach((callback) => callback(self.error));
            }, 0)
        }
    }
    try {
        fn(resolve, reject);
    } catch (e) {
        reject(e);
    }
}

function resolvePromise(bridgepromise, x, resolve, reject) {
    if (bridgepromise === x) {
        return reject(new TypeError('Circular reference'));
    }

    let called = false;
    if (x instanceof MyPromise) {
        if (x.status === PENDING) {
            x.then(y => {
                resolvePromise(bridgepromise, y, resolve, reject);
            }, error => {
                reject(error);
            });
        } else {
            x.then(resolve, reject);
        }
    } else if (x != null && ((typeof x === 'object') || (typeof x === 'function'))) {
        try {
            let then = x.then;
            if (typeof then === 'function') {
                then.call(x, y => {
                    if (called) return;
                    called = true;
                    resolvePromise(bridgepromise, y, resolve, reject);
                }, error => {
                    if (called) return;
                    called = true;
                    reject(error);
                })
            } else {
                resolve(x);
            }
        } catch (e) {
            if (called) return;
            called = true;
            reject(e);
        }
    } else {
        resolve(x);
    }
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
    const self = this;
    let bridgePromise;
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : value => value;
    onRejected = typeof onRejected === "function" ? onRejected : error => { throw error };
    if (self.status === FULFILLED) {
        return bridgePromise = new MyPromise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let x = onFulfilled(self.value);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            }, 0);
        })
    }
    if (self.status === REJECTED) {
        return bridgePromise = new MyPromise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let x = onRejected(self.error);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            }, 0);
        });
    }
    if (self.status === PENDING) {
        return bridgePromise = new MyPromise((resolve, reject) => {
            self.onFulfilledCallbacks.push((value) => {
                try {
                    let x = onFulfilled(value);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
            self.onRejectedCallbacks.push((error) => {
                try {
                    let x = onRejected(error);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
        });
    }
}
MyPromise.prototype.catch = function(onRejected) {
    return this.then(null, onRejected);
}

MyPromise.deferred = function() {
    let defer = {};
    defer.promise = new MyPromise((resolve, reject) => {
        defer.resolve = resolve;
        defer.reject = reject;
    });
    return defer;
}
try {
    module.exports = MyPromise
} catch (e) {}
```



## 实现 all、race、resolve、reject

```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function MyPromise(fn) {
    const self = this;
    self.value = null;
    self.error = null;
    self.status = PENDING;
    self.onFulfilledCallbacks = [];
    self.onRejectedCallbacks = [];

    function resolve(value) {
        if (value instanceof MyPromise) {
            return value.then(resolve, reject);
        }
        if (self.status === PENDING) {
            setTimeout(() => {
                self.status = FULFILLED;
                self.value = value;
                self.onFulfilledCallbacks.forEach((callback) => callback(self.value));
            }, 0)
        }
    }

    function reject(error) {
        if (self.status === PENDING) {
            setTimeout(function() {
                self.status = REJECTED;
                self.error = error;
                self.onRejectedCallbacks.forEach((callback) => callback(self.error));
            }, 0)
        }
    }
    try {
        fn(resolve, reject);
    } catch (e) {
        reject(e);
    }
}

function resolvePromise(bridgepromise, x, resolve, reject) {
    if (bridgepromise === x) {
        return reject(new TypeError('Circular reference'));
    }

    let called = false;
    if (x instanceof MyPromise) {
        if (x.status === PENDING) {
            x.then(y => {
                resolvePromise(bridgepromise, y, resolve, reject);
            }, error => {
                reject(error);
            });
        } else {
            x.then(resolve, reject);
        }
    } else if (x != null && ((typeof x === 'object') || (typeof x === 'function'))) {
        try {
            let then = x.then;
            if (typeof then === 'function') {
                then.call(x, y => {
                    if (called) return;
                    called = true;
                    resolvePromise(bridgepromise, y, resolve, reject);
                }, error => {
                    if (called) return;
                    called = true;
                    reject(error);
                })
            } else {
                resolve(x);
            }
        } catch (e) {
            if (called) return;
            called = true;
            reject(e);
        }
    } else {
        resolve(x);
    }
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
    const self = this;
    let bridgePromise;
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : value => value;
    onRejected = typeof onRejected === "function" ? onRejected : error => { throw error };
    if (self.status === FULFILLED) {
        return bridgePromise = new MyPromise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let x = onFulfilled(self.value);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            }, 0);
        })
    }
    if (self.status === REJECTED) {
        return bridgePromise = new MyPromise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let x = onRejected(self.error);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            }, 0);
        });
    }
    if (self.status === PENDING) {
        return bridgePromise = new MyPromise((resolve, reject) => {
            self.onFulfilledCallbacks.push((value) => {
                try {
                    let x = onFulfilled(value);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
            self.onRejectedCallbacks.push((error) => {
                try {
                    let x = onRejected(error);
                    resolvePromise(bridgePromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
        });
    }
}
MyPromise.prototype.catch = function(onRejected) {
    return this.then(null, onRejected);
}

MyPromise.all = function(promises) {
    return new MyPromise(function(resolve, reject) {
        let result = [];
        let count = 0;
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(function(data) {
                result[i] = data;
                if (++count == promises.length) {
                    resolve(result);
                }
            }, function(error) {
                reject(error);
            });
        }
    });
}

MyPromise.race = function(promises) {
    return new MyPromise(function(resolve, reject) {
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(function(data) {
                resolve(data);
            }, function(error) {
                reject(error);
            });
        }
    });
}

MyPromise.resolve = function(value) {
    return new MyPromise(resolve => {
        resolve(value);
    });
}

MyPromise.reject = function(error) {
    return new MyPromise((resolve, reject) => {
        reject(error);
    });
}

module.exports = MyPromise
```



## 实现 promiseify方法

```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

function MyPromise(fn) {
  const self = this;
  self.value = null;
  self.error = null;
  self.status = PENDING;
  self.onFulfilledCallbacks = [];
  self.onRejectedCallbacks = [];

  function resolve(value) {
    if (value instanceof MyPromise) {
      return value.then(resolve, reject);
    }
    if (self.status === PENDING) {
      setTimeout(() => {
        self.status = FULFILLED;
        self.value = value;
        self.onFulfilledCallbacks.forEach(callback => callback(self.value));
      }, 0);
    }
  }

  function reject(error) {
    if (self.status === PENDING) {
      setTimeout(function() {
        self.status = REJECTED;
        self.error = error;
        self.onRejectedCallbacks.forEach(callback => callback(self.error));
      }, 0);
    }
  }
  try {
    fn(resolve, reject);
  } catch (e) {
    reject(e);
  }
}

function resolvePromise(bridgepromise, x, resolve, reject) {
  if (bridgepromise === x) {
    return reject(new TypeError('Circular reference'));
  }

  let called = false;
  if (x instanceof MyPromise) {
    if (x.status === PENDING) {
      x.then(
        y => {
          resolvePromise(bridgepromise, y, resolve, reject);
        },
        error => {
          reject(error);
        }
      );
    } else {
      x.then(resolve, reject);
    }
  } else if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      let then = x.then;
      if (typeof then === 'function') {
        then.call(
          x,
          y => {
            if (called) return;
            called = true;
            resolvePromise(bridgepromise, y, resolve, reject);
          },
          error => {
            if (called) return;
            called = true;
            reject(error);
          }
        );
      } else {
        resolve(x);
      }
    } catch (e) {
      if (called) return;
      called = true;
      reject(e);
    }
  } else {
    resolve(x);
  }
}

MyPromise.prototype.then = function(onFulfilled, onRejected) {
  const self = this;
  let bridgePromise;
  onFulfilled =
    typeof onFulfilled === 'function' ? onFulfilled : value => value;
  onRejected =
    typeof onRejected === 'function'
      ? onRejected
      : error => {
          throw error;
        };
  if (self.status === FULFILLED) {
    return (bridgePromise = new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onFulfilled(self.value);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      }, 0);
    }));
  }
  if (self.status === REJECTED) {
    return (bridgePromise = new MyPromise((resolve, reject) => {
      setTimeout(() => {
        try {
          let x = onRejected(self.error);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      }, 0);
    }));
  }
  if (self.status === PENDING) {
    return (bridgePromise = new MyPromise((resolve, reject) => {
      self.onFulfilledCallbacks.push(value => {
        try {
          let x = onFulfilled(value);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
      self.onRejectedCallbacks.push(error => {
        try {
          let x = onRejected(error);
          resolvePromise(bridgePromise, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
    }));
  }
};
MyPromise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected);
};

MyPromise.all = function(promises) {
  return new MyPromise(function(resolve, reject) {
    let result = [];
    let count = 0;
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(
        function(data) {
          result[i] = data;
          if (++count == promises.length) {
            resolve(result);
          }
        },
        function(error) {
          reject(error);
        }
      );
    }
  });
};

MyPromise.race = function(promises) {
  return new MyPromise(function(resolve, reject) {
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(
        function(data) {
          resolve(data);
        },
        function(error) {
          reject(error);
        }
      );
    }
  });
};

MyPromise.resolve = function(value) {
  return new MyPromise(resolve => {
    resolve(value);
  });
};

MyPromise.reject = function(error) {
  return new MyPromise((resolve, reject) => {
    reject(error);
  });
};

MyPromise.promisify = function(fn) {
  return function() {
    var args = Array.from(arguments);
    return new MyPromise(function(resolve, reject) {
      fn.apply(
        null,
        args.concat(function(err) {
          err ? reject(err) : resolve(arguments[1]);
        })
      );
    });
  };
};

module.exports = MyPromise;

```

