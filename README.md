# firebase-micropython-esp32
Firebase implementation for Micropython on ESP WROOM 32. Not tested on other platforms running micropython but should work.

This is a very basic and ported version of https://github.com/shariq/firebase-python for micropython. This is WIP. All the basic features like:
```
1. GET
2. POST
3. PUSH
3. PATCH
```
work fine.


Requires `urequests` and `usseclient`. `urequests` is a part of micropython already. I have added a ported version of `sseclient` removing other dependencies not available in micropython. No other dependencies have to be added.

## Connect to wifi

```
run the startup.py file to connect to WiFi. Pass ssid and password as arguments. Put ssid under single/double quotes
if it contains spaces.

>>> execfile('startup.py ssid password')
```

Now you are good to go.

## get and put

`get` gets the value of a Firebase at some URL, `put` writes or replaces data at a Firebase path.

```python
>>> import ufirebase as firebase
>>> URL = 'lucid-lychee'  # see note on URLs at the bottom of documentation
>>> print firebase.get(URL)  # this is an empty Firebase
None

>>> firebase.put(URL, 'tell me everything')  # can take a string
>>> print firebase.get(URL)
tell me everything

>>> firebase.put(URL, {'lucidity': 9001})  # or a dictionary
>>> print firebase.get(URL)
{u'lucidity': 9001}

>>> firebase.put(URL, {'color': 'red'})  # replaces old value
>>> print firebase.get(URL)
{u'color': u'red'}

>>> print firebase.get(URL + '/color')
red
```



## push

`push` pushes data to a list on a Firebase path. This is the same as `patch`ing with an incrementing key, with Firebase taking care of concurrency issues.

```python
>>> import ufirebase as firebase
>>> URL = 'bickering-blancmanges'
>>> print firebase.get(URL)
None

>>> firebase.push(URL, {'color': 'pink', 'jiggliness': 'high'})
>>> firebase.get(URL)
{
  u'-JyAXHX9ZNBh7tPPja4w': {u'color': u'pink', u'jiggliness': u'high'}
}

>>> firebase.push(URL, {'color': 'white', 'jiggliness': 'extreme'})
>>> firebase.get(URL)
{
  u'-JyAXHX9ZNBh7tPPja4w': {u'color': u'pink', u'jiggliness': u'high'},
  u'-JyAXHX9ZNBh7tPPjasd': {u'color': u'white', u'jiggliness': u'extreme'}
}
```



## patch

`patch` adds new key value pairs to an existing Firebase, without deleting the old key value pairs.

```python
>>> import ufirebase as firebase
>>> URL = 'tibetan-tumbleweed'
>>> print firebase.get(URL)
None

>>> firebase.patch(URL, {'taste': 'tibetan'})
>>> print firebase.get(URL)
{u'taste': u'tibetan'}

>>> firebase.patch(URL, {'size': 'tumbly})  # patching does not overwrite
>>> print firebase.get(URL)
{u'taste': u'tibetan', u'size': u'tumbly'}
```



## subscriber (WIP. This won't work)

`subscriber` takes a URL and callback function and calls the callback on every update of the Firebase at URL.

```python
>>> import firebase
>>> from pprint import pprint  # function which pretty prints objects
>>> URL = 'clumsy-clementine'
>>> S = firebase.subscriber(URL, pprint)  # pprint will be called on all Firebase updates
>>> S.start()  # will get called with initial value of URL, which is empty
(u'put', {u'data': None, u'path': u'/'})

>>> firebase.put(URL, ';-)')  # will make S print something
(u'put', {u'data': u';-)', u'path': u'/'})

>>> firebase.put(URL, {'status': 'mortified'})  # continuing from above
(u'put', {u'data': {u'status': u'mortified'}, u'path': u'/'})
>>> firebase.patch(URL, {'reason': 'blushing'})  # same data, different method
(u'patch', {u'data': {u'reason': u'blushing'}, u'path': u'/'})

>>> firebase.put(URL + '/color', 'red')
(u'put', {u'data': u'red', u'path': u'/color'})

>>> S.stop()
```



## URLs (WILL BE REMOVED in the upcoming versions for reducing any code latency) 
All URLs are internally converted to the apparent Firebase URL. This is done by the `firebaseURL` method.

```python
>>> import firebase

>>> print firebase.firebaseURL('bony-badger')
https://bony-badger.firebaseio.com/.json

>>> print firebase.firebaseURL('bony-badger/bones/humerus')
https://bony-badger.firebaseio.com/bones/humerus.json

>>> print firebase.firebaseURL('bony-badger.firebaseio.com/')
https://bony-badger.firebaseio.com/.json
```

## TO-DO:

1. Remove whatever latency possible.
2. Async Subscription to changes is WIP
