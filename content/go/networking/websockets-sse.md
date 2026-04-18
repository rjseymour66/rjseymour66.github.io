+++
title = 'Websockets and Server-sent events'
linkTitle = "Websockets and SSEs"
date = '2025-09-07T10:17:27-04:00'
weight = 30
draft = false
+++

## Websockets

A websocket lets you create real-time and bidirectional persistent connections to deliver data efficiently. These connections remove some of the overhead and delays in the HTTP/TCP layer (such as polling).

The websocket protocol keeps TCP connections active and can multiplex incoming messages to multiple clients as they arrive.

## Server-sent events

A server-sent event (SSE, or `EventSource` in JS) is a long-lived HTTP connection that allows communication from the server to client only (unidirectional). SSEs are good for messages like notifications.

Note that there is a per-browser limit on the number of open connections to a single domain. This is across all windows and tabs.