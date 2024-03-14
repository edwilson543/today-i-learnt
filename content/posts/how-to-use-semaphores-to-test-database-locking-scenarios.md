+++
title = 'How to use semaphores to test database locking scenarios'
date = 2023-11-28T18:14:59Z
tags = ["Python", "Threading", "Database", "Testing"]
+++

Testing code that acquires a database lock is tricky because it requires
more than one database transaction to be open simultaneously.

The approach I learnt below opens a transaction on two separate threads and uses
a semaphore [(a type of synchronization primitive)][semaphore-python-docs] to 
co-ordinate between the threads. 

The flow is as follows:
- Both threads open a transaction and attempt to acquire the lock
- The 'happy' thread gets there first and locks the resource
- The 'happy' thread holds the lock until the 'unhappy' thread fails
- The 'unhappy' thread fails to acquire the lock and errors
- On error, the 'unhappy' tells the 'happy' thread to release the lock

Note that this approach is deterministic. On the other hand using something like 
`time.sleep` to control how long the 'happy' thread holds the lock is not and
therefore is prone to flake.

```python
import threading
from typing import Any, Callable

from django.db import close_old_connections, transaction


def attempt_acquiring_lock_in_separate_transactions(
    *, acquire_lock: Callable[[], Any], timeout: int = 20
) -> list[Exception]:
    errors: list[Exception] = []
    semaphore = threading.Semaphore(value=0)

    def locking_attempt() -> None:
        try:
            with transaction.atomic():
                acquire_lock()
                # Happy thread: hold the database lock until the semaphore can be acquired.
                # The initial value is 0 (the minimum), so the unhappy thread must release
                # (increment) it first.
                if not semaphore.acquire(blocking=True, timeout=timeout):
                    raise TimeoutError("Happy thread waited too long for unhappy thread to fail.")
        except Exception as exc:
            # Unhappy thread: Release the semaphore, so it can be acquired by the happy thread.
            semaphore.release()
            errors.append(exc)

        close_old_connections()

    threads = [threading.Thread(target=locking_attempt) for _ in range(0, 2)]
    for thread in threads:
        thread.start()
    for thread in threads:
        thread.join()

    return errors
```

[semaphore-python-docs]: https://docs.python.org/3/library/threading.html#semaphore-objects