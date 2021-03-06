eXpMPP
======

XMPP notifications for psychology experiments

Installation
======

## With pip (recommended)

The latest stable binaries are available via pip.  Simply run `pip install expmpp --user`

## From github

1. `git clone https://github.com/louist87/expmpp.git`
2. `cd expmpp`
3. `python setup.py develop --user`

Usage
======

## Setting up a client

In order to begin receiving notifications, we must first initialize a client.
This should be done **exactly once** in your application and the resultant `Client` instance can then be imported by various submodules.

```python
from expmpp.client import Client

my_listeners = ['mylistener@domain.com']  # ID of the account being notified
client = Client('myuser@domain.com', 'mypassword', listeners=my_listeners)
```

## Sending notifications

Once you've initialized your client, you can begin sending arbitrary notifications.

```python
client.notify('This is a test.')
```

## Monitoring functions

Sometimes it is useful to be notified when a specific function returns.  A common use-case is to send a notificaiton to the experimentor when the function responsible for running an experimental block has completed.  This use-case motivates the following example:

```python
@client.monitor("Block Complete")
def run_block():
    # logic to run the block
```

When the function `run_block` returns, eXpMPP will send a notification containing the text `Block Complete`.

It is often desirable to provide information about the return value of the monitored function.  By default, `Client.monitor` attempts to fill a [python-formatted string](https://docs.python.org/2/library/stdtypes.html#str.format) with the return value of the monitored function.  Thus,

```python
@client.monitor("Block {0} Complete")
def run_block():
    # logic to run block
    return block_num
```

is expected to return a string such as `Block 1 Complete`, assuming `run_block` returns an integer.

For functions that return multiple values (or iterable containers), the `unpack` flag, when set to `True`, will attempt to map each variable in the returned container to its respective placeholder.  For instance:

```python
@client.monitor("Subject {0}, Block {1} Complete", unpack=True)
def run_block():
    # logic to run block
    return sub_num, block_num
```

The above example is expected to return a string such as `Subject 1, Block 3 Complete`.

If the function returns a dictionary, setting `unpack=True` will map the the values of the dictionary to named placeholders as follows:

```python
@client.monitor("Subject {sub}, Block {block} Complete")
def run_block():
    # logic to run block
    return {'sub': sub_num, 'block': block_num}
```

The above example is expected to return a string similar to the one in the preceding example.

### Transforming output for notification

On occasion, a function will return a value that is either non-human-readable or whose default formatting is sinfully ugly.  For these cases, a function can be passed to the `transformer` keyword argument, which allows a developer to transform the output before notification.  Note that the `transformer` parameter does **not** change the function's final return value; it only changes what gets sent over the wire.

```python
def check_err(ret_val):
    if ret_val is None:
        return "Block complete.  No errors."
    else:
        return "Error:  {0}".format(ret_val)


@client.monitor('{0}', transformer=check_err)
def run_block():
    # logic to run block
    return ret_val
```
