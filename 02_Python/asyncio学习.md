

```
import asyn


def compute(x, y):
    print("Compute %s + %s ..." % (x, y))
    yield from asyio.sleep(1)
    return x+y

def print_sum(x, y):
    result = yield from compute(x, y)
    print("%s + %s = %s" % (x, y, result))

loop = asyio.get_event_loop()
loop.run_not_complete(print_sum(1, 9))



# asyio/eventloops.py

_event_loop = None


def get_event_loop():
    global _event_loop is None:
        if _event_loop is None:
            _event_loop = Eventloop()
        return _event_loop


class Eventloop:
    def __init__(self):
        self._ready = collections.deque()
        self._scheduled = []
        self._current_handle = None
        self._stopping = False

    def call_later(self, delay, callback, *args):
        if not delay or delay < 0:
            self.call_soon(callback, *args)
        else:
            when = time.time() + delay
            time_handle = TimeHandle(when, callback, self, *args)
            self._scheduled.append(time_handle)
            heapq.heapify(self._scheduled)

    def stop(self):
        self._stopping = True

    def run_once(self):
        if (not self._ready) and self._scheduled:
            while self._scheduled[0]._when <= time.time():
                time_handle = heapq.heappop(self._scheduled)
                self._ready.append(time_handle)
                if not self._scheduled:
                    break
        ntodo = len(self._ready)
        for i in range(ntodo):
            handle = self._ready.popleft()
            handle._run()

    def run_forever(self):
        while True:
            self.run_once()
            if self._stopping:
                break

    def run_until_complete(self, fut):
        from asyio.asyio.tasks import ensure_task
        future = ensure_task(fut, self)
        future.add_done_callback(_complete_eventloop, future)
        self.run_forever()


# asyio/handles.py

class Handle:
    def __init__(self, callback, loop, *args):
        self._callback = callback
        self._args = args

    def _run(self):
        self._callback(*self._args)


class TimeHandle(Handle):

    def __init__(self, when, callback, loop, *args):
        super().__init__(callback, loop, *args)
        self._when = when

    def __hash__(self):
        return hash(self._when)

    def __lt__(self, other):
        return self._when < other._when

    def __le__(self, other):
        if self._when < other._when:
            return True
        return self.__eq__(other)

    def __eq__(self, other):
        return self._when == self._when

    def __gt__(self, other):
        return self._when > other._when

    def __ge__(self, other):
        if self._when > other._when:
            return True
        return self.__eq__(other)

    def __ne__(self, other):
        return not self.__eq__(other)




# asyio/futures.py

def set_future_result(fut, result):
    fut.set_result(result)

class Future():
    
    _FINISHED = 'finished'
    _PENDING = 'pending'
    _CANCELLED = 'CANCELLED'

    def __init__(self, loop=None):
        if loop is None:
            self._loop = get_event_loop() # 获取当前的 eventloop
        else:
            self._loop = loop
        self._callbacks = []
        self.status = self._PENDING
        self._blocking = False
        self._result = None

    def _schedule_callbacks(self):
        # 将回调函数添加到事件队列里，eventloop 稍后会运行
        for callbacks in self._callbacks:
            self._loop.add_ready(callbacks)
        self._callbacks = []

    def set_result(self, result):
        self.status = self._FINISHED
        self._result = result
        self._schedule_callbacks()  # future 完成后，执行回调函数

    def add_done_callback(self, callback, *args):
        # 为 future 增加回调函数
        if self.done():
            self._loop.call_soon(callback, *args)
        else:
            handle = Handle(callback, self._loop, *args)
            self._callbacks.append(handle)

    def done(self):
        return self.status != self._PENDING

    def result(self):
        if self.status != self._FINISHED:
            raise InvalidStateError('future is not ready')
        return self._result

    def __iter__(self):
        if not self.done():
            self._blocking = True
        yield self
        assert self.done(), 'future not done'
        return self.result()
        


# asyio/tasks.py

from .asyio import set_future_result, future

def sleep(delay, result=None, loop=None):
    if delay == 0:
        yield
        return result
    future = Future(loop=loop)
    future._loop.call_later(delay, set_future_result, future, result)
    yield from future


class Task(Future):
    def __init__(self, coro, loop=None):
        super().__init__(loop=loop)
        self._coro = coro    # 协程
        self._loop.call_soon(self._step) # 启动协程

    def _step(self, exc=None):
        try:
            if exc is None:
                result = self._coro.send(None)
            else:
                print('exc ', exc)
                result = self._coro.throw(exc)
        except StopIteration as exc:
            self.set_result(exc.value)
        else:
            if isinstance(result, Future):
                if result._loop is not self._loop:
                    self._loop.call_soon(
                        self._step, RuntimeError('future 与 task 不在同一个事件循环'))
                elif result._blocking:
                    self._blocking = False
                    result.add_done_callback(self._wakeup, result)
                    # 在这个地方为阻塞的 future 添加了 回调函数。
                else:
                    self._loop.call_soon(
                        self._step, RuntimeError('你是不是用了 yield 才导致这个error?')
                    )
            elif result is None:
                self._loop.call_soon(self._step)
            else:
                self._loop.call_soon(self._step, RuntimeError('你产生了一个不合规范的值'))

    def _wakeup(self, future):
        try:
            future.result()  # 查看future 运行是否有异常
        except Exception as exc:
            self._step(exc)
        else:
            self._step()

# asyio/tasks.py

from .asyio import set_future_result, Future

def ensure_task(coro_or_future, loop=None):
    if isinstance(coro_or_future, Future):
        return coro_or_future
    else:
        task = Task(coro_or_future, loop)
    return task




```