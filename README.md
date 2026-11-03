# Ticko-BugSearched "testnet.ticko.xyz" OR "ticko.xyz"
Ran check HTTP headers of target
Ran fetch HTML from target
Ran read endpoints JS file
Ran find base URLs in endpoints.js
Ran extract base URLs from endpoints.js · Exited with code 1
Ran extract var S from endpoints.js
Ran analyze trade.js endpoints
Ran inspect refDataSlice
Ran find scripts in index.html · Exited with code 1
Created inspect_ticko.js
Ran run endpoint inspection script
Created download_assets.js
Ran download key client JS files
Created analyze_ws.js
Ran run WS analysis script
Created search_ws_details.js
Ran search WS details
Created inspect_cm_details.js
Ran inspect ChannelManager details
Created test_ws.js
Ran test live WebSocket connection · Exited with code 1
Writing test_ws.js
Edited test_ws.js
Ran run native WebSocket test
Edited test_ws.js
Ran test account WebSocket connection
Created inspect_auth.js
Ran run inspect_auth script
Created search_eip.js
Ran search EIP712 across bundles
Checked task Search EIP712 across bundles
Created inspect_guard.js
Ran run inspect_guard
Created inspect_trade_details.js
Ran run inspect trade details
Created inspect_ordersign.js
Ran inspect order signature logic
Created trace_i.js
Ran run trace_i
Created find_ihook.js
Ran run find_ihook
Created find_idef.js
Ran run find_idef
Created find_fndecl.js
Ran find enclosing function
Created find_comp.js
Ran run find_comp
Created find_ks.js
Ran run find_ks
Created find_useagentsign.js
Ran run find_useagentsign
Created search_placeorder.js
Ran search PlaceOrder in channelMgr
Created find_placeorder_asset.js
Ran find PlaceOrder definition file
Created inspect_guard_full.js
Ran run inspect guard full
Created inspect_ve.js
Ran run inspect_ve
Created find_btc_symbol.js
Ran find BTC symbol metadata · Exited with code 1
Edited find_btc_symbol.js
Ran inspect symbol object structure
Created test_market_data.js
Ran test market data endpoints

### 1. Environment & Network Configuration

Passive inspection of client assets on [`testnet.ticko.xyz/trade?market=BTC-USDT`](https://testnet.ticko.xyz/trade?market=BTC-USDT) (Release Build `841ff003`) reveals the production testnet topology:

* **Frontend Origin**: `https://testnet.ticko.xyz`
* **Static Assets / CDN**: `https://static.testnet.ticko.xyz/unix-web/testnet/841ff003/assets/`
* **Core REST API Base**: `https://api.testnet.ticko.xyz`
* **Public Market WebSocket**: `wss://ws.testnet.ticko.xyz/v1/public/market`
* **User Account WebSocket**: `wss://ws.testnet.ticko.xyz/v1/public/account`
* **Third-Party Identity & Infrastructure**:
  * Privy Embedded Wallet: `https://auth.privy.io` (`cmrpqmzna00df0ck1faafc4t5`)
  * WalletConnect Relay: `https://relay.walletconnect.com` (`66728424be29047ea0b067afc25dcb1a`)
  * Telemetry / Faro Collector: `https://faro.unixin.org/collect`

---

### 2. Evidence-Based API Endpoint Inventory

All endpoints listed below are extracted directly from client routing bundles (`endpoints-C2dULkrx.js`, `EnableTradingGuard-DFXwJ04d.js`, `trade-DBJrzWDT.js`) and verified via passive initial request capture on Testnet.

| Category | Endpoint Path | Method | Observed Status Code | Authentication / Authorization Required | Verified Payload / Query Parameters & Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Market Data** | `/v1/market/symbols` | `GET` | `200 OK` | Public | Returns market configuration parameters (lot size, tick size, leverage). BTC-USDT is registered as `symbol_id: 20001`. |
| **Market Data** | `/v1/market/coins` | `GET` | `200 OK` | Public | Returns supported asset metadata (e.g., USDT `coin_id: 1001`, BTC `coin_id: 10001`). |
| **Market Data** | `/v1/market/ref-data` | `GET` | `200 OK` | Public | Returns dictionary mapping symbol and coin IDs to display names. |
| **Market Data** | `/v1/market/tickers` | `GET` | `200 OK` | Public | Returns 24h rolling ticker statistics across active symbols. |
| **Market Data** | `/v1/market/fee-rates` | `GET` | `200 OK` | Public | Returns VIP tier maker and taker fee schedules. |
| **Market Data** | `/v1/market/l2_book` | `GET` | `200 OK` | Public | Parameter: `symbol_id` (e.g. `20001`). Returns L2 depth asks/bids. Without parameter, returns `400 Bad Request` (`code: 10006`). |
| **Market Data** | `/v1/market/trades` | `GET` | `200 OK` | Public | Parameter: `symbol_id`. Returns recent public trade executions. |
| **Market Data** | `/v1/market/klines` | `GET` | `200 OK` | Public | Parameters: `symbol_id`, `interval` (e.g. `1m`). Returns OHLC candlestick history. |
| **Market Data** | `/v1/market/funding-rate/history` | `GET` | `200 OK` | Public | Parameter: `symbol_id`. Returns historical perp funding rate intervals and mark prices. |
| **Authentication & Agent Keys** | `/v1/account/approve-agent` | `POST` | `400 Bad Request` (without signature) | EIP-712 Signature | Registers an ephemeral trading key (`agentAddress`) for an account. Payload requires `signer_address`, `agent_address`, `valid_days`, `signature: {r, s, v}`. |
| **Authentication & Agent Keys** | `/v1/account/revoke-agent` | `POST` | *Declared* | EIP-712 Signature | Revokes an active delegated agent key. |
| **Authentication & Agent Keys** | `/v1/account/renew-agent` | `POST` | *Declared* | EIP-712 Signature | Renews time-to-live for an active agent key. |
| **Account & Margin** | `/v1/account/balance` | `GET` | `400 Bad Request` (`code: 10006`) | Requires `address` & Agent/Bearer token | Returns collateral equity, free margin, and unrealized PnL. Requires `address` parameter. |
| **Account & Margin** | `/v1/account/positions` | `GET` | `400 Bad Request` (`code: 10006`) | Requires `address` & Agent/Bearer token | Returns open positions, entry price, liquidation price, margin mode. Requires `address` parameter. |
| **Account & Margin** | `/v1/account/positions/history` | `GET` | *Declared* | Requires `address` & Agent/Bearer token | Historical closed positions and realized PnL. |
| **Account & Margin** | `/v1/account/leverage` | `GET` / `POST` | *Declared* | Agent Signature (`ActionTag: 11`) | Reads/sets market leverage bracket. |
| **Account & Margin** | `/v1/account/position-mode` | `GET` / `POST` | *Declared* | Agent Signature (`ActionTag: 10`) | Sets position mode (`one_way` vs `hedge`). |
| **Account & Margin** | `/v1/account/isolated-margin` | `POST` | *Declared* | Agent Signature (`ActionTag: 15`) | Adjusts margin allocated to isolated positions. |
| **Account & Margin** | `/v1/account/settings` | `GET` | *Declared* | Address token | Account configuration preferences. |
| **Trade Execution** | `/v1/trade/orders` | `GET` | `400 Bad Request` (`code: 10006`) | Requires `address` parameter | Query active orders for connected address. |
| **Trade Execution** | `/v1/trade/orders` | `POST` | *Declared* | Agent Signature (`ActionTag: 7`) | Submits new limit/market/conditional order. Signed by local agent key. |
| **Trade Execution** | `/v1/trade/orders/cancel` | `POST` | *Declared* | Agent Signature (`ActionTag: 8`) | Cancels specific order by `order_id` or `client_order_id`. |
| **Trade Execution** | `/v1/trade/orders/cancel-all` | `POST` | *Declared* | Agent Signature (`ActionTag: 9`) | Cancels all open orders across a market or account. |
| **Trade Execution** | `/v1/trade/orders/modify` | `POST` | *Declared* | Agent Signature (`ActionTag: 12`) | Modifies price/size of existing resting order. |
| **Trade Execution** | `/v1/trade/orders/chase` | `POST` | *Declared* | Agent Signature (`ActionTag: 13`) | Adjusts order to follow top-of-book (BBO chase). |
| **Trade Execution** | `/v1/trade/orders/batch` | `POST` | *Declared* | Agent Signature (`ActionTag: 17`) | Submits multiple orders in a single request. |
| **Trade Execution** | `/v1/trade/orders/batch/cancel`| `POST` | *Declared* | Agent Signature (`ActionTag: 16`) | Cancels batch of orders. |
| **Trade Execution** | `/v1/trade/orders/cancel-all-after` | `POST` | *Declared* | Agent Signature | Dead-man's switch / heartbeat timeout cancellation. |
| **Trade Execution** | `/v1/trade/executions` | `GET` | *Declared* | Requires `address` parameter | Historical order fill logs for user. |

---

### 3. WebSocket Infrastructure & Connection Status

The application operates two independent WebSocket channels:

| WebSocket Endpoint | Handshake Status | Frame Protocol & Heartbeat | Observed Subscribed Channels | Verification Status |
| :--- | :--- | :--- | :--- | :--- |
| `wss://ws.testnet.ticko.xyz/v1/public/market` | `101 Switching Protocols` | JSON framing. Client sends `{"op":"ping"}`; server replies `{"op":"pong","time":<epoch_ms>}`. Re-ping interval: 30s. | `bbo`, `trade`, `kline`, `l2_book`, `ticker`, `all_tickers`, `coin_price`, `explorer_block`, `explorer_txs` | **Live Verified**: Connection established, pong returned, clean close. |
| `wss://ws.testnet.ticko.xyz/v1/public/account` | `101 Switching Protocols` | JSON framing. Ping/pong supported. Subscription format: `{"id":<id>, "op":"subscribe", "args":[{"channel":"<chan>", "address":"<addr>"}]}`. | `order`, `execution`, `position`, `balance`, `settings` | **Live Verified**: Handshake succeeds, pong returned. Subscriptions require user address. |

---

### 4. Client-Side Trading & Security Mechanics (Verified from Source)

#### A. BTC-USDT Market Specification (Extracted from `/v1/market/symbols`)
* `symbol_id`: `20001`
* `base_coin_id`: `10001` (BTC), `quote_coin_id`: `1001` (USDT), `settle_coin_id`: `1001` (USDT)
* `min_size`: `0.001` BTC, `max_size`: `10000` BTC, `qty_precision`: `3`, `qty_step`: `0.001`
* `price_precision`: `1`, `price_step`: `0.1`
* `default_leverage`: `20x`, `max_leverage`: `150x`

#### B. Order Construction Payload
When placing an order to `POST /v1/trade/orders`, the client prepares:
```json
{
  "symbol_id": 20001,
  "is_buy": true,
  "order_type": "limit",
  "client_order_id": "<1-32 chars alphanumeric>",
  "quantity": "0.001",
  "position_side": "both",
  "margin_mode": "cross",
  "time_in_force": "gtc",
  "price": "77350.0",
  "flags": []
}
```

#### C. Session / Agent Key Authorization Mechanism (`EnableTradingGuard`)
1. **Delegation / Session Key Registration**:
   * Client generates a local ephemeral keypair (`agentAddress`).
   * Main wallet signs an EIP-712 `ApproveAgent` message:
     * Domain: `{ name, version, chainId, verifyingContract }`
     * Message: `{ signerAddress, tickoChain, agentAddress, authorizedAddress, validDays, label, nonce, expiresAfter }`
   * POSTed to `/v1/account/approve-agent`.
2. **Per-Action Agent Signing**:
   * Trading actions use numeric action tags:
     `{ Deposit: 2, PlaceOrder: 7, CancelOrder: 8, CancelAll: 9, SetPositionMode: 10, SetLeverage: 11, ModifyOrder: 12, ChaseOrder: 13, UpdateMargin: 15, BatchCancel: 16, BatchOrder: 17, BatchModify: 18 }`
   * The action payload is deterministically serialized (sorted keys, escaped characters), concatenated with the 1-byte `actionTag`, and hashed: `actionHash = keccak256([actionTag, canonicalJsonBytes])`.
   * The local ephemeral agent signs an EIP-712 `Agent` structure containing `{ signerAddress, targetAddress, actionHash, nonce, expiresAfter }`.
   * The signature `{ r, s, v }`, `nonce`, and `expires_after` are attached to the API request body.

---

### 5. Separation of Verified Facts vs. Unverified Assumptions

| Component | Verified from Application Code & Traffic | Unverified / Pending Authorized Testing |
| :--- | :--- | :--- |
| **Endpoints** | Exact paths, HTTP methods, and parameter names confirmed via bundles and passive queries. | Internal backend microservice routing behind Cloudflare. |
| **Authentication** | EIP-712 `ApproveAgent` + local `Agent` key signature delegation scheme verified from bytecode. | Whether the backend strictly validates `nonce` replay and `expiresAfter` boundaries on every action tag. |
| **Account Isolation**| Requests without `address` return HTTP 400 (`code: 10006`). | Whether server enforces strict cross-account isolation (IDOR protection) when an agent signs for a mismatched `targetAddress`. |
| **WebSocket** | Server handshake, URLs, and JSON ping/pong frames live verified. | Authorization enforcement on private channels (`wss://ws.testnet.ticko.xyz/v1/public/account`) when subscribing to foreign addresses. |
