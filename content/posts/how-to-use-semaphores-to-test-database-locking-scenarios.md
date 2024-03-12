+++
title = 'How to use semaphores to test database locking scenarios'
date = 2023-11-28T18:14:59Z
draft = true
tags = ["Python", "Threading", "Database"]
+++

Testing code that acquires a database lock is tricky because it requires
more than one database transaction to be open simultaneously.

- To write up
