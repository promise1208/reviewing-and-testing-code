# Embedded Connectivity And IoT Software Review

Load this reference for MQTT, MQTT-SN, HTTP, HTTPS, WebSocket, CoAP, LwM2M, TLS, DTLS, TCP, UDP, sockets, DNS, DHCP, SNTP, Wi-Fi software, Bluetooth Classic, BLE, GAP, GATT, ATT, SMP, HCI, Matter, Thread, Zigbee, LoRaWAN, NB-IoT, LTE-M, cellular modems, AT commands, PPP, pairing, bonding, provisioning, cloud client, or IoT protocol changes. Review code and repository-supported software tests only; do not require a live network, radio, device, HIL, emulator, simulator, or fake transport.

## Transport And Connection State

- Trace create/init, DNS or address resolution, connect, authentication, ready, send/receive, partial I/O, timeout, disconnect, reconnect, cancellation, shutdown, destroy, and repeated init/deinit states.
- Check blocking versus nonblocking calls, callback or task context, socket/error return handling, partial reads/writes, EOF, retry and exponential-backoff bounds, jitter arithmetic, keepalive, stale callbacks, duplicate connection events, and teardown races.
- Verify buffer ownership, zero-copy lifetime, queue/backpressure, maximum frame/message sizes, fragmentation/reassembly, byte order, length arithmetic, integer overflow, credentials, and cleanup after every failure.

## MQTT And MQTT-SN

- Validate fixed header, remaining-length decoding, packet type/flags, topic and payload lengths, UTF-8 constraints, packet identifiers, properties for MQTT 5, and malformed/truncated/concatenated packet handling.
- Review QoS 0/1/2 state transitions, PUBACK/PUBREC/PUBREL/PUBCOMP handling, duplicate and retransmit flags, in-flight limits, retained messages, last will, clean start/session expiry, subscribe/unsubscribe acknowledgements, and reconnect resubscription policy.
- Check keepalive and ping timing, tick wraparound, offline queues, persistence ownership, duplicate delivery/idempotency expectations, broker rejection, authentication failure, disconnect reason codes, and bounded recovery.

## HTTP, HTTPS, And WebSocket

- Validate method, scheme, host, port, path/query encoding, request headers, authorization, content type, content length, chunked transfer, body limits, streaming callbacks, and partial response parsing.
- Handle status classes, redirects and redirect limits, retry safety by method, connection reuse, close-delimited bodies, malformed headers, conflicting lengths, unsupported encoding, timeout, cancellation, and server truncation.
- For WebSocket, check upgrade validation, masking direction, opcode, fragmentation, control-frame limits, ping/pong, close handshake, payload bounds, UTF-8 where required, and reconnect state.

## TLS, DTLS, And Credentials

- Verify certificate-chain and hostname validation, SNI, time-validity dependency, trust-store selection, protocol-version floor, cipher configuration, entropy/RNG error handling, session resumption, and close-notify behavior from code and configuration.
- Check certificate, private-key, token, PSK, and Wi-Fi credential lifetime, storage API errors, buffer clearing where implemented, log redaction, error-message exposure, provisioning updates, and rollback/partial-write handling.
- Treat insecure verification bypass, hard-coded production secrets, disabled hostname checks, unbounded handshake retry, and acceptance of expired or untrusted certificates as high-severity findings.

## Bluetooth Classic And BLE

- Trace controller/host initialization, enable/disable, advertising, scanning, connection, security, service discovery, data transfer, disconnect, reconnect, and teardown state machines.
- For GAP/GATT/ATT, check address and identity handling, advertising/scan data lengths, UUID and handle lookup, characteristic permissions, read/write/prepare/execute behavior, CCCD state, notifications/indications, MTU-dependent fragmentation, and callback lifetime.
- Review pairing, bonding, SMP keys, passkey/numeric comparison state, authorization, bond database updates, duplicate peers, privacy addresses, timeout, cancellation, and failed-pair cleanup.
- For HCI/L2CAP/RFCOMM or profile code, check packet length, credits, channels, queues, ownership, partial completion, disconnect races, and error propagation. Never claim RF quality, range, coexistence, antenna, or over-the-air interoperability.

## Other IoT Protocols

- For CoAP, validate token/message ID, confirmable retransmission, duplicate detection, block-wise transfer, observe lifecycle, option parsing, timeout, and DTLS integration.
- For LwM2M, validate object/resource IDs, read/write/execute permissions, registration/update/deregister state, bootstrap credentials, observe/notify state, serialization bounds, and reconnect behavior.
- For provisioning, cloud SDKs, OTA-control protocols, and device twins, check schema/version compatibility, idempotency, replay/duplicate events, command authorization, offline state, bounded queues, cancellation, and persistent-state recovery.

## Matter, Thread, Zigbee, LoRaWAN, And Cellular

- For Matter, Thread, and Zigbee, review commissioning/join state, fabrics or network credentials, endpoint/cluster/attribute IDs, TLV or frame bounds, transaction IDs, duplicate handling, subscriptions/reporting, access control, persistent state, leave/reset, and callback ownership.
- For LoRaWAN, review OTAA/ABP configuration, join/rejoin state, region/channel/data-rate constants, frame counters and persistence, confirmed-message retries, duty-cycle arithmetic represented in code, payload bounds, key lifetime, and rollback behavior. Do not infer RF compliance or coverage.
- For NB-IoT, LTE-M, cellular modems, AT commands, and PPP, review command/response and unsolicited-result-code parsing, echo and line termination, partial/concatenated responses, state transitions, APN and credential handling, SIM/PIN errors, registration and PDP-context state represented in code, timeout, cancellation, reconnect, and stale responses.

## Software Evidence

Run repository-supported cross-builds, warnings, static analysis, parser/state-machine tests, and existing protocol tests. Pure-logic tests may cover malformed frames, state transitions, retry arithmetic, bounds, and security policy without adding fake transports. Report exact commands, configurations, covered cases, and unexecuted network/radio behavior; do not infer live connectivity or interoperability.
