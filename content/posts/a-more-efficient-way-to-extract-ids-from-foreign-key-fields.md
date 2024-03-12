+++
title = 'A more efficient way to extract IDs from Foreign Key fields'
date = 2024-02-19T18:34:46Z
draft = true
tags = ["Python", "Django", "Database"]
+++

Given a Django foreign key field `related_model`, I previously used:
```python
related_model_id = model_instance.related_model_id
```
and
```python
related_model_id = model_instance.related_model.id
```
interchangeably.

I learnt that the latter results in an additional DB query for all of 
`related_model`'s fields versus the former. 
This is because `related_model_id` is just the name of the DB column Django
creates to represent the foreign key, and is therefore available when
`model_instance` is first loaded into memory.
