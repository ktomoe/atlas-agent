# Rules of pseudo code examples
* `event` denotes the variable used to access each container within an event.
* `ii` denotes the index of each object.
* `num_{object_name}` denotes the number of objects in an event.

The following retrieves the ii-th muon pT in the event.
```python
muon_pt = event['muon_container.pt'][ii]
```

Pseudo code in these references is a **specification of the selection logic for
one object** — it states *what* is required of the ii-th object. It is not an
implementation. The code must be optimized according to the size of the dataset.
The following is an example of how to vectorize the operation.

## Vectorized counterpart
The same selection over a whole array, once the dataset makes the loop too slow.
The translation is mechanical.
~~~
# pseudo code -- one muon                    # vectorized -- every muon at once
mu_pt = event['...pt'][ii]                   mu_pt = array['...pt']
mu_q = event['...quality'][ii] & 0x3         mu_q = array['...quality'] & 0x3
if mu_q > 1 or mu_pt < 5000:                 mu_pass = (mu_q <= 1) & (mu_pt >= 5000)
    continue                                 array['...pt'][mu_pass]
~~~
