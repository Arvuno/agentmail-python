# Agentmail Python Library

[![fern shield](https://img.shields.io/badge/%F0%9F%8C%BF-Built%20with%20Fern-brightgreen)](https://buildwithfern.com?utm_source=github&utm_medium=github&utm_campaign=readme&utm_source=https%3A%2F%2Fgithub.com%2Fagentmail-to%2Fagentmail-python)
[![pypi](https://img.shields.io/pypi/v/agentmail)](https://pypi.python.org/pypi/agentmail)

The Agentmail Python library provides convenient access to the Agentmail APIs from Python.

## Table of Contents

- [Installation](#installation)
- [Reference](#reference)
- [Usage](#usage)
- [Async Client](#async-client)
- [Exception Handling](#exception-handling)
- [Advanced](#advanced)
  - [Access Raw Response Data](#access-raw-response-data)
  - [Retries](#retries)
  - [Timeouts](#timeouts)
  - [Custom Client](#custom-client)
- [Contributing](#contributing)
- [Websockets](#websockets)

## Installation

```sh
pip install agentmail
```

## Quick Start

```python
import os
import asyncio
from agentmail import AgentMail

client = AgentMail(api_key=os.environ.get("AGENTMAIL_API_KEY"))

async def main() -> None:
    await client.inboxes.create()

asyncio.run(main())
```

A full reference for this library is available [here](https://github.com/agentmail-to/agentmail-python/blob/HEAD/./reference.md).

## Usage

Instantiate and use the client with the following:

```python
from agentmail import AgentMail

client = AgentMail(
    api_key="<token>",
)

client.inboxes.create()
```

## Async Client

The SDK also exports an `async` client so that you can make non-blocking calls to our API. Note that if you are constructing an Async httpx client class to pass into this client, use `httpx.AsyncClient()` instead of `httpx.Client()` (e.g. for the `httpx_client` parameter of this client).

```python
import asyncio

from agentmail import AsyncAgentMail

client = AsyncAgentMail(
    api_key="<token>",
)

async def main() -> None:
    await client.inboxes.create()

asyncio.run(main())
```

## Exception Handling

When the API returns a non-success status code (4xx or 5xx response), a subclass of the following error
will be thrown.

```python
from agentmail.core.api_error import ApiError

try:
    client.inboxes.create(...)
except ApiError as e:
    print(e.status_code)
    print(e.body)
```

## Advanced

### Access Raw Response Data

The SDK provides access to raw response data, including headers, through the `.with_raw_response` property.
The `.with_raw_response` property returns a "raw" client that can be used to access the `.headers` and `.data` attributes.

```python
from agentmail import AgentMail

client = AgentMail(...)
response = client.inboxes.with_raw_response.create(...)
print(response.headers)  # access the response headers
print(response.status_code)  # access the response status code
print(response.data)  # access the underlying object
```

### Retries

The SDK is instrumented with automatic retries with exponential backoff. A request will be retried as long
as the request is deemed retryable and the number of retry attempts has not grown larger than the configured
retry limit (default: 2).

A request is deemed retryable when any of the following HTTP status codes is returned:

- [408](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408) (Timeout)
- [429](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) (Too Many Requests)
- [5xx](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500) (Server Error)

If a retry limit is configured (default: 2) the request will be retried until the limit is reached.

### Retries Configuration

You can configure the number of retries and the delay between retries on any client:

```python
from agentmail import AgentMail

client = AgentMail(
    api_key="<token>",
    max_retries=0,  # default: 2
)
```

The SDK uses an exponential backoff strategy with jitter. For more information on the details of the strategy see the [code](./agentmail/core/client.py).

### Timeouts

The default timeout is 60 seconds. You can configure it on the client:

```python
from agentmail import AgentMail

client = AgentMail(
    api_key="<token>",
    timeout=20.0,  # default: 60
)
```

Or on a per-request basis:

```python
from agentmail import AgentMail

client = AgentMail(api_key="<token>")
client.inboxes.list(timeout=5.0)
```

### Custom Client

You can customize the client by passing your own `httpx.Client` or `httpx.AsyncClient` to the `http_client` parameter:

```python
import httpx
from agentmail import AgentMail

client = AgentMail(
    api_key="<token>",
    http_client=httpx.Client(
        proxy="http://my代理人.com:1234",
        timeout=5.0,
    ),
)
```

## Contributing

Contributions are welcome! Please see the contributing guide for more details.

## Websockets

Websocket support is available for real-time communication. For more details, see the [Websockets documentation](https://docs.agentmail.to).