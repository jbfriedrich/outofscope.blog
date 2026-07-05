---
title: PyPi Service Degradation
date: 2020-12-22T19:00:00
tags:
  - python
  - development
languages:
  - en
---

Just tried to search for some Django modules in `pip` and got a very nasty surprise.

```python
ERROR: Exception:
Traceback (most recent call last):
  File "/usr/local/lib/python3.8/site-packages/pip/_internal/cli/base_command.py", line 224, in _main
    status = self.run(options, args)
  File "/usr/local/lib/python3.8/site-packages/pip/_internal/commands/search.py", line 62, in run
    pypi_hits = self.search(query, options)
  File "/usr/local/lib/python3.8/site-packages/pip/_internal/commands/search.py", line 82, in search
    hits = pypi.search({'name': query, 'summary': query}, 'or')
  File "/usr/local/lib/python3.8/xmlrpc/client.py", line 1109, in __call__
    return self.__send(self.__name, args)
  File "/usr/local/lib/python3.8/xmlrpc/client.py", line 1450, in __request
    response = self.__transport.request(
  File "/usr/local/lib/python3.8/site-packages/pip/_internal/network/xmlrpc.py", line 46, in request
    return self.parse_response(response.raw)
  File "/usr/local/lib/python3.8/xmlrpc/client.py", line 1341, in parse_response
    return u.close()
  File "/usr/local/lib/python3.8/xmlrpc/client.py", line 655, in close
    raise Fault(**self._stack[0])
xmlrpc.client.Fault: <Fault -32500: "RuntimeError: PyPI's XMLRPC API has been temporarily disabled due to unmanageable load and will be deprecated in the near future. See https://status.python.org/ for more information.">
```

[Python's status site](https://status.python.org) has more nasty details:

> **Update** — Dec 15, 20:59 UTC
> The XMLRPC Search endpoint is still disabled due to ongoing request volume. As of this update, there has been no reduction in inbound traffic to the endpoint from abusive IPs and we are unable to re-enable the endpoint, as it would immediately cause PyPI service to degrade again. We are working with the abuse contact at the owner of the IPs and trying to make contact with the maintainers of whatever tool is flooding us via other channels.
> **Monitoring** — Dec 14, 17:46 UTC
> With the temporary disabling of XMLRPC we are hoping that the mass consumer that is causing us trouble will make contact. Due to the huge swath of IPs we were unable to make a more targeted block without risking more severe disruption, and were not able to receive a response from their abuse contact or direct outreach in an actionable time frame.
> **Update** — Dec 14, 17:30 UTC
> Due to the overwhelming surges of inbound XMLRPC search requests (and growing) we will be temporarily disabling the XMLRPC search endpoint until further notice.
> **Identified** — Dec 14, 15:09 UTC
> We've identified that the issue is with excess volume to our XLMRPC search endpoint that powers `pip search` among other tools. We are working to try to identify patterns and prohibit abusive clients to retain service health.
> **Investigating** — Dec 14, 09:41 UTC
> PyPI's search backends are experiencing an outage causing the backends to timeout and fail, leading to degradation of service for the web app. Uploads and installs are currently unaffected but logged in actions and search via the web app and API access via XMLRPC are currently experiencing partial outages. 

Damn, had to happen one day I guess. Single point of failure, infrastructure provided by volunteers and goodwill. Hope they can resolve this soon.
