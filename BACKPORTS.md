# Backports to the RokuOS curl 8_19_1 branch

This file lists, period by period, the upstream curl commits brought into this
branch after its release base, and the fork-only commits they needed. The
newest period is at the top; each later integration adds a new period above
the previous one and leaves the older periods as they are.

Kinds:

- **pick**: the upstream commit applies unchanged;
- **backport**: adapted to this release; the commit message carries
  `Backport notes:` saying what changed and why;
- **fork**: a fork-only commit, such as a build or test repair the
  backports needed.

Picks and backports keep the upstream author and end with a
`(cherry picked from commit ...)` trailer naming the upstream commit.

Release base: curl 8.19.0 (`curl-8_19_0`, `8c908d2d0a`, 2026-03-11). This
history starts at `b181da67ac`, the 8_19_1 branch before these periods; the
changes up to there are not listed here.

<!--
To add a period, copy this block above the newest period and fill it in.
Upstream commits are curl's; dates are commit dates (YYYY-MM-DD).

## Period N: YYYY-MM-DD, <short title>

| | |
|---|---|
| Fork commits | `<previous tip>..<new tip>` (N commits: P picks, B backports, L fork-only) |
| Upstream reviewed | from `<hash>` (`<tag>`, YYYY-MM-DD) to `<hash>` (`<tag>`, YYYY-MM-DD) |
| Verified | <builds, test suites, application tests> |

<one or two sentences: what the period was about>

#### <Area>

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `<fork hash>` | `<upstream hash>` | YYYY-MM-DD | pick/backport/fork | <subject> — <note> |
-->

## Period 2: 2026-11-18, HTTP proxy fixes

| | |
|---|---|
| Fork commits | `50d9787a6e..e4851af6e8` (6 commits: 0 picks, 5 backports, 1 fork-only) |
| Upstream reviewed | from `c951368579` (`last_reviewed_5`, 2026-06-12) to `c9cae58a8b` (`last_reviewed_6`, 2026-11-17), the same range as period 1 |
| Verified | Windows builds (OpenSSL, mbedTLS); Linux runtests (preset, mbedTLS, all-protocols) at the baseline failures, plus the new proxy tests 750, 1827, 2824, 5032; pytest unchanged; CurlTestApplication passed |

No new upstream commits: this period revisited proxy fixes that period 1 had
skipped as unused. RokuOS does use HTTP proxies, through the `http_proxy`
environment variable that libcurl reads itself.

#### HTTP proxy

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `fc7bd68a4b` | `9f121f75a7` | 2026-09-24 | backport | cf-h1-proxy: reset chunk state on new request — A chunked 407 reply no longer makes the next CONNECT reply be read as chunked |
| `346bfefecf` | `c1a1a2968e` | 2026-09-24 | backport | cf-h1-proxy: stricter status line parsing — A later header line can no longer override the proxy's status code |
| `e1d2b13b4b` | `8eb9b4abd0` | 2026-11-11 | backport | proxy: authentication header handling — Proxy credentials are no longer sent past a CONNECT tunnel |
| `271cf7e6af` | `0613e0ec32` | 2026-11-12 | backport | http-proxy: reconnect on non-chunked CONNECT error framing — A reply with both Content-Length and Transfer-Encoding is framed correctly before an auth retry |
| `29950e11c2` | `12938b611c` | 2026-11-14 | backport | http-proxy: honor timeouts while a CONNECT body arrives — A proxy that keeps sending a 407 body could stall the transfer past its timeout |

#### Fork-only

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `e4851af6e8` | — | — | fork | tests: expect error 56 from a failed CONNECT in 1827 and 2824 — The release returns 56 for a failed tunnel; the ported tests expected upstream's later 7 |

## Period 1: 2026-11-12 to 2026-11-18, first integration after the release

| | |
|---|---|
| Fork commits | `b181da67ac..50d9787a6e` (97 commits: 34 picks, 54 backports, 9 fork-only) |
| Upstream reviewed | from `c951368579` (`last_reviewed_5`, 2026-06-12) to `c9cae58a8b` (2026-11-17) |
| Verified | Windows builds (OpenSSL, mbedTLS); Linux runtests (preset, mbedTLS, all-protocols) at the baseline failures, including HTTP/3 tests 2500-2503; pytest 672 passing, 91 over HTTP/3; lib1560; CurlTestApplication passed |

Every upstream commit in the range got a Take or Skip decision; the
picks and backports below are the Takes. Must-have fixes first: security
fixes, crashes, leaks and hangs in code the RokuOS build compiles (FILE,
HTTP and HTTPS; OpenSSL and mbedTLS; HTTP/2 and HTTP/3). Six commits come
from before the range, taken as prerequisites or for the URL API: `3f06e27502`,
`67bf021e97`, `9135294115`, `831a151484`, `d74c0ada4e`, `04afd16076`.

#### Security (CVE) fixes

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `660418c59b` | `90325ff044` | 2026-08-05 | pick | http2: make server push transfers inherit share from parent — CVE-2026-18924 (Low) |
| `8c0939998c` | `7ea37abc6a` | 2026-08-25 | pick | openssl: avoid conn reuse if provider is used — CVE-2026-80229 (Low) |
| `dd8b6aba6d` | `c3c70440e8` | 2026-08-26 | pick | setopt: make NULL `CURLOPT_AWS_SIGV4` disable aws-sigv4 auth — CVE-2026-86409 (Low) |
| `92ad8906dd` | `72352daba0` | 2026-08-27 | pick | connection reuse: check SSL configs when doing a scheme upgrade — CVE-2026-86411 (Low) |
| `ee46201794` | `43029e2ec3` | 2026-09-09 | pick | url: ntlm/negotiate, prohibit connection reuse on follows — CVE-2026-86413 (Low) |
| `c958d68099` | `445aa87e13` | 2026-09-16 | pick | cookie: accept slightly longer lines in cookie jars — CVE-2026-92386 (Low) |
| `b3c6fbbdc4` | `8b0744af5b` | 2026-10-12 | backport | multi: adjust the cleanup order in multi_done — CVE-2026-92392 (High) |
| `59344fae89` | `d7546809d5` | 2026-09-13 | backport | mime: do curl_mime_free() non-recursively — CVE-2026-92391 (Low) |
| `8e02b0c7c0` | `5267ed859d` | 2026-08-25 | backport | openssl: no server cert is only okay if also not pinned — CVE-2026-80230 (Low) |
| `0e6e9d702f` | `4f6aa41a01` | 2026-08-27 | backport | cookie: improve TAB handling — CVE-2026-80255 (Low) |
| `810c6e6b87` | `2956af6c74` | 2026-09-04 | backport | HTTP-CONNECT: do not react to 401 responses — CVE-2026-86414 (Low) |
| `980c48fefb` | `6ded3e5944` | 2026-09-05 | backport | cookie: ignore individual cookie errors when loading — CVE-2026-86412 (Low) |
| `580d5d3734` | `7a6bd027d0` | 2026-09-21 | backport | getinfo: make sure CURLINFO_EFFECTIVE_URL does not contain creds — CVE-2026-94266 (Low) |
| `f5cf8ce588` | `d37d79df1e` | 2026-10-09 | backport | url: CURLOPT_INTERFACE fix connection reuse match — CVE-2026-107929 (Low) |

#### OpenSSL

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `2394c491fa` | `93eb74a610` | 2026-11-08 | pick | openssl: do full peer checks when the offered session is not resumed |
| `9306808381` | `3acbc6d5cd` | 2026-11-09 | pick | openssl: disable SSL_MODE_AUTO_RETRY |
| `8d8f7c35ff` | `90345a3e8e` | 2026-11-10 | pick | openssl: allow OCSP response to be up to 4 days old |
| `34b8431145` | `543adc4de0` | 2026-08-31 | backport | openssl: avoid strlen() on the data from OpenSSL |
| `d11b9f0d00` | `c53420e427` | 2026-09-04 | backport | ossl_verifyhost: remove assumption of null termination of ASN1_STRING. |
| `1c2e1e7b8a` | `6d303dfc11` | 2026-10-30 | backport | openssl: stop I/O after fatal shutdown errors |
| `fbf49040be` | `3c60b05098` | 2026-10-12 | backport | openssl: support X509 store caching when CApath is configured |
| `3664b85243` | `2c5f9c8ac0` | 2026-10-17 | backport | openssl: avoid CA caching for providers |
| `e03906bd46` | `55652ad0de` | 2026-10-07 | pick | openssl: hardening |
| `e8ff127efc` | `e8bd2e746f` | 2026-11-16 | pick | openssl: stop shutdown after a fatal SSL_read |

#### mbedTLS

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `06d7e1437b` | `5b10939ef7` | 2026-06-30 | pick | mbedtls: replace `memset()` with `psa_hash_operation_init()` |
| `462ff003f0` | `76847b05e3` | 2026-09-08 | pick | mbedtls: fix ticket handling |
| `65827fc43c` | `bc7863a097` | 2026-09-24 | pick | mbedtls: resend length check |
| `2ff3cd6d7e` | `26fdb92c0a` | 2026-08-09 | backport | mbedtls: enforce verifyhost when verifypeer is disabled |
| `61d1d50a40` | `67bf021e97` | 2026-05-07 | pick | mbedtls: null terminate the private key blob |
| `9338c21102` | `c00a3021e4` | 2026-10-09 | backport | mbedtls: pass correct length for PEM/DER key blobs to mbedtls_pk_parse_key |

#### TLS layer

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `5f48aef322` | `c2ce945ac0` | 2026-08-25 | pick | vtls_scache: use case sensitive path match |
| `b3c99786bf` | `717ebe4d0a` | 2026-09-02 | backport | vtls: fix Curl_cert_hostcheck when the pattern is not a C string |
| `0a6325fa02` | `0e0cb78834` | 2026-11-07 | pick | hostcheck: do not strip trailing dot from dNSName/CN |

#### URL parsing

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `58018a4199` | `3f06e27502` | 2026-03-18 | pick | urlapi: make dedotdotify handle leading dots correctly |
| `f0584602fe` | `9135294115` | 2026-05-15 | backport | urlapi: deny hostnames with more than one trailing dot |
| `fa7fdad559` | `04afd16076` | 2026-06-09 | backport | urlapi: URL decode hostname before IP address normalization |
| `2729fce739` | `6ae66b31d9` | 2026-09-14 | backport | urlapi: reject spaces and control characters in zone id |
| `f48c155cc8` | `f4caaa2354` | 2026-09-14 | backport | urlapi: reject more junk from the zone id |
| `e7f6cb12f2` | `a7023cafb8` | 2026-10-29 | backport | urlapi: make zoneids starting with 25 to work |
| `687612e0aa` | `2bc780243f` | 2026-10-29 | backport | urlapi: return a blank user blank even with CURLU_URLENCODE |
| `f415f1e825` | `13e84a6945` | 2026-09-22 | backport | urlapi: make curl_url_dup() copy the guessed_scheme flag |
| `61f0398afc` | `d611ca7e3e` | 2026-10-24 | backport | urlapi: do not pass a NULL path to printf for file URLs |
| `4f850169b1` | `831a151484` | 2026-05-15 | backport | urlapi: consume trailing dots after IPv4 numerical addresses — The release's own test 1560 already expected this |
| `dc52b04da4` | `d74c0ada4e` | 2026-05-17 | backport | urlapi: prevent a terminal `.0x` component to normalize IPv4 — Required follow-up to 831a151484 |

#### Connections

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `8dd5ae5dc4` | `ca26e728a6` | 2026-08-13 | pick | conncache: don't assume curl_off_t increment wrap-around |
| `098a10438a` | `24dfe3169c` | 2026-10-09 | pick | cfilters: fail a send that reports more written than requested |
| `1fab8b9206` | `02214d98f7` | 2026-07-12 | backport | conncache: apply multi limits to transfers using a shared pool |
| `a3b5262787` | `820c014578` | 2026-07-17 | backport | cfilters: fix event-based connection shutdown |
| `05e161a12a` | `146698eac0` | 2026-10-01 | backport | conncache: never discard a connection still in use |
| `a9503c5812` | `6968ebf100` | 2026-10-28 | backport | connection close: additional checks |

#### HTTP/2

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `138a330bd6` | `72986f2aa6` | 2026-09-01 | pick | http2: remove assert in ingress processing |
| `adb30670f3` | `3f5b46c153` | 2026-09-01 | pick | h2proxy: remember goaway "last_stream_id" |
| `e9ccaf569a` | `3a397f7a05` | 2026-09-16 | pick | http2: update a refused stream's error after GOAWAY |
| `5d0aaba39a` | `790a219284` | 2026-09-25 | pick | http2: fix reset after 1xx response |
| `3090dfebfa` | `5057074f70` | 2026-10-01 | pick | http2: limit trailers and headers count |
| `480392eb04` | `c4b341c485` | 2026-10-19 | pick | http2: do not let a rate limit enlarge the stream window |
| `af3f1f683b` | `9315d9c4e4` | 2026-09-16 | backport | http2: initialize URL handle for server pushes |
| `d806d5dc25` | `11df32bc8e` | 2026-09-17 | backport | http2: don't send a CURLE_SEND_ERROR after a valid http2 response |

#### HTTP core

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `2c7044076d` | `94390cdb02` | 2026-10-01 | pick | http_chunks: limit trailers size |
| `f5b55142e0` | `b09d38d759` | 2026-11-10 | pick | http: init 'leading_unfold' for each request |
| `d4dd195c76` | `a36384ab94` | 2026-06-26 | backport | http: trim custom header name before the Authorization drop |
| `08c12b8f0f` | `3f684481b8` | 2026-11-14 | backport | http: reject malformed header terminators |
| `aab0ae5304` | `a70bdea474` | 2026-11-13 | backport | http: detach borrowed headers from mime part on error — Use-after-free after a failed MIME post |

#### Multi / event loop

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `9b7f044bd1` | `e9a2b0dc72` | 2026-06-25 | pick | multi_ev: refresh sock entry after remove callback — Use-after-free |
| `316bfa6180` | `ef362b78ac` | 2026-09-14 | backport | multi-event: fix timeout of pending transfers |
| `1532c4f3b9` | `191dc2ad9d` | 2026-10-21 | backport | multi: undo the counters when curl_multi_add_handle fails |
| `00412a1333` | `11d0b20213` | 2026-10-07 | backport | multi: is_finished() fix |
| `b14ed86835` | `2dee5ea294` | 2026-09-10 | backport | mulit_ev: remove socket references on forget — Socket identity confusion on fd reuse |

#### Options / share

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `f2982aa3c6` | `f7d4e11f4b` | 2026-06-22 | pick | setopt: return error if received `curl_blob->data` is NULL |
| `0ad8caf660` | `a8b9f53118` | 2026-10-19 | backport | setopt: keep a paused transfer paused when the speed limit is changed — Paused transfers could hang. Caused a throttling stall on reused handles, fixed by 67e892c141 |
| `1c23320b5d` | `fe2d3591c3` | 2026-08-28 | backport | share unlink: forget connection |
| `922a16d48b` | `10bf036282` | 2026-10-07 | backport | share link: forget connection |

#### Transfer engine

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `5fb4426937` | `8940c1436b` | 2026-09-24 | pick | cw-out: recheck callbacks for each call |
| `91570d7a84` | `acf8a65be7` | 2026-10-20 | pick | transfer: drop a pending read rewind when a new transfer starts |
| `5c88b5f3e3` | `eb6d1e098e` | 2026-06-15 | backport | ratelimits: use minimal burst rate — Rate-limited HTTP/2 and HTTP/3 transfers could stall |
| `72768891d7` | `eda23d0405` | 2026-11-04 | backport | transfer: add Curl_xfer_try_send() — Uploads could hang when only the end-of-stream marker was blocked |
| `e130de31c4` | `f29c5d3fa0` | 2026-10-20 | backport | cw-pause: only pass EOS with the last flushed piece |
| `19ec165c30` | `2ba56e30f8` | 2026-11-16 | pick | file: report download read errors — A file:// read error no longer ends as success with truncated data |
| `6a518d6c4f` | `a75e50419b` | 2026-11-12 | backport | transfer: add `never_retry` bit — A failed HTTP/2 pushed transfer is never retried as a new request |
| `50d9787a6e` | `c3f9ef13f6` | 2026-07-13 | backport | mime: reject CR and LF in mail part name and filename — Stops a CR/LF in a name from splitting the Content-Disposition header (mail and FORMESCAPE) |

#### DNS

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `a26bd4b0cc` | `c3ae9ef822` | 2026-07-13 | backport | hostip: only cache negative resolves for authoritative answers — A transient DNS failure no longer blocks a host for about 30 s |

#### HTTP/3 (ngtcp2)

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `4c7c9aa22a` | `caff91bf3c` | 2026-08-27 | backport | ngtcp2: avoid NULL deref in cf_ngtcp2_send |
| `8ccba004b4` | `9e5292a003` | 2026-09-02 | backport | ngtcp2: shrink max sendbuf — Per-stream upload buffer capped at 640 KB instead of 10 MB |
| `70bb0dd0bb` | `ed1bd03520` | 2026-11-02 | backport | vquic: access CMSG_DATA() only if enough data |
| `1c26984780` | `d342e8e87b` | 2026-11-17 | pick | quic: when sending without GSO, check lengths — Endless send loop when GSO is disabled |
| `68f4cb8b4d` | `eebb23fd41` | 2026-11-12 | backport | ngtcp2: fix busy pool on send — Packet assembly could spin forever |
| `4da2a244eb` | `bddb2ddafb` | 2026-11-16 | backport | ngtcp2: honor stream packet coalescing contract |
| `84c448583e` | `c4334a57ec` | 2026-11-12 | backport | http3: deliver response trailers as trailers, do not process as headers — Set-Cookie or Location in an HTTP/3 trailer no longer takes effect |

#### Fork-only

| Commit | Upstream | Upstream date | Kind | Change |
|---|---|---|---|---|
| `d559a84255` | — | — | fork | share: add Curl_share_easy_link/unlink() for backported code |
| `84b0266e07` | — | — | fork | 8_19_1: fix the Linux build and the test build after backports |
| `effda7a096` | — | — | fork | cookie: reject TAB in cookie names again |
| `fef2a28a9b` | — | — | fork | tests: initialize the result in lib2118 |
| `6897ee5927` | — | — | fork | share: cast to CURLSH in Curl_share_easy_link() |
| `0a332928f4` | — | — | fork | url: fix NTLM/Negotiate reuse check on redirects — The CVE-2026-86413 backport crashed tests 89, 90, 1087 and 1100 |
| `4753523f0e` | — | — | fork | tests: add the libtest tool to test3234 |
| `3273be1628` | — | — | fork | tests: fix lib1560 expectations for this release |
| `67e892c141` | — | — | fork | setopt: start the rate limiter fresh unless the transfer is paused — Fixes the ThrottleReaderTest regression from 0ad8caf660, also confirmed in upstream master |
