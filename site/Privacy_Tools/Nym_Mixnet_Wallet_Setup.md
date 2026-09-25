<a href="https://github.com/zechub/zechub/edit/main/site/Privacy_Tools/Nym_Mixnet_Wallet_Setup.md" target="_blank">
  <img src="https://img.shields.io/badge/Edit-blue" alt="Edit Page"/>
</a>

# Route Zcash Wallet Traffic Over the Nym Mixnet

> Last verified: September 2026

Zcash shielded transactions protect transaction data on-chain, but a wallet still has to communicate over the internet. Network observers can potentially learn metadata such as your IP address, when your wallet connects, and which infrastructure it contacts.

NymVPN can add a separate layer of network privacy by routing traffic through Nym's network before it reaches public Zcash infrastructure.

This guide focuses on the safest broadly compatible approach: **run the wallet through the system-level NymVPN tunnel in Mixnet mode**. It also explains why Nym's app-specific SOCKS5 mode should not be assumed to work with every Zcash wallet.

For a broader introduction to VPNs and decentralized VPNs, see [VPN & dVPN](./VPN_and_DVPN.md).

## What Nym adds — and what it does not

A shielded Zcash payment and a network privacy tool solve different problems:

- **Zcash shielded pools** protect transaction details on-chain.
- **NymVPN Mixnet mode** makes it harder to associate your home or mobile IP address and packet timing with the destination service.
- The public service you contact should see the Nym exit gateway's IP rather than your own.

Nym's Mixnet mode currently sends traffic through an entry gateway, three mix-node layers with randomized delays, and an exit gateway. Nym also generates cover traffic to make timing correlation harder.

Nym does **not** protect against a compromised device, a malicious wallet build, exposed recovery phrases, or information you reveal through transparent Zcash activity or third-party accounts.

## Recommended setup: system-wide NymVPN Mixnet mode

This path requires no special proxy support inside the wallet.

### 1. Install NymVPN

Download NymVPN only from Nym's official site or the platform's official app store.

Official NymVPN information and downloads:

- https://nym.com/
- https://nym.com/blog/nymvpn-v2026.11

NymVPN supports Android, iOS, Linux, Windows, and macOS.

### 2. Connect in Mixnet mode

Open NymVPN and select **Mixnet mode**.

NymVPN also offers a faster two-hop mode. That mode can still hide your home IP from the destination, but it does not use the same multi-hop mixing and timing-delay design as Mixnet mode.

Wait until NymVPN reports that the connection is established before opening or refreshing your wallet.

### 3. Leave the wallet on its normal network settings

For most Zcash wallets, no wallet-side proxy configuration is required when the operating system is already routing traffic through NymVPN.

Open the wallet normally and allow it to sync.

On platforms where NymVPN exposes split tunneling, make sure the wallet is **included in the protected tunnel**, not placed on a bypass/exclusion list.

### 4. Verify the tunnel before sending funds

A simple check verifies that the operating system's default internet path changed:

1. Disconnect NymVPN.
2. Visit an IP-checking service or run a command such as:

   ```bash
   curl https://api.ipify.org
   ```

3. Record the visible IP address.
4. Connect NymVPN in Mixnet mode.
5. Repeat the check.

The second public IP should be different.

This verifies the system tunnel, not the wallet implementation itself. For desktop wallets, an advanced user can additionally inspect active connections with the operating system's network monitor while NymVPN is connected.

## Zodl (formerly Zashi)

Zodl has its own built-in Tor support in addition to whatever network tunnel the operating system uses.

Zodl's official support documentation states that Tor Protection can route transaction submission, transaction-data retrieval, exchange-rate requests, and third-party API connections over Tor.

Current Zodl Tor instructions:

**More → Advanced Features → Beta: Tor Protection → Enable → Save changes**

Source:

- https://support.zodl.com/article/17-enabling-tor-protection

Zodl also offers the Tor option during wallet recovery/synchronization, and its support documentation warns that Tor can make synchronization slower:

- https://support.zodl.com/article/13-recovering-your-zodl-wallet

### NymVPN plus Zodl

If you use the **system-wide NymVPN Mixnet tunnel**, Zodl does not need a Nym-specific setting.

For a simple and auditable setup, use one network-privacy layer deliberately rather than assuming that stacking Tor and NymVPN always improves privacy. Running Zodl's Tor client inside a NymVPN tunnel can add latency and complexity, and ZecHub has not independently established that this combination provides a meaningful additional benefit.

## YWallet and its successor Zkool

YWallet is a legacy wallet line. Its actively maintained successor, **Zkool**, describes itself as the successor to YWallet and supports Tor proxying and onion services for its Zcash server connections.

Current project:

- https://github.com/mladenmarkov/zkool

For Nym, the recommended approach remains the **system-level NymVPN Mixnet tunnel**, because this does not depend on wallet-specific proxy implementation details.

Do not assume that a field labeled "Tor proxy" is automatically interchangeable with Nym's SOCKS5 mode. A wallet may make Tor-specific assumptions, including onion-service handling.

If you still use an older YWallet build, verify its current networking options and maintenance status before relying on wallet-specific proxy settings.

## About Nym's dApp / wallet SOCKS5 mode

NymVPN also has a dApp/wallet mode that exposes a SOCKS5 path through the mixnet.

Nym's current documentation describes this mode primarily for wallets that can be pointed at a compatible RPC endpoint through SOCKS5, with Ethereum wallets used as the main example:

- https://nym.com/blog/nymvpn-v2026.2
- https://nym.com/blog/nymvpn-dapp-mode

This can be useful for software that explicitly supports a generic SOCKS5 proxy, but **do not configure a Zcash wallet this way unless that wallet documents compatible SOCKS5 proxy support for its lightwalletd or full-node connections**.

For a Zcash wallet without documented generic SOCKS5 support, use the system-level NymVPN tunnel instead.

## Verifying wallet traffic more carefully

For higher assurance on desktop:

1. Connect NymVPN in Mixnet mode.
2. Start the wallet.
3. Confirm that sync begins successfully.
4. Use the operating system's network monitor to observe the wallet process.
5. Confirm there is no intentional split-tunnel rule excluding the wallet.
6. If practical, temporarily disconnect NymVPN and confirm that the wallet's network behavior changes as expected.

Avoid posting screenshots of wallet addresses, balances, transaction IDs, IP addresses, or recovery material while troubleshooting.

## Performance and timeout trade-offs

Mixnets intentionally add delay. Nym's documentation explains that Mixnet mode introduces randomized packet delays and cover traffic, while newer Mixnet Tuning controls let users trade some anonymity margin for responsiveness.

Current Nym explanation:

- https://nym.com/docs/network/mixnet-mode/traffic-flow
- https://nym.com/blog/mixnet-tuning

For Zcash light wallets, this can affect:

- initial synchronization,
- large catch-up syncs,
- transaction-history queries,
- RPC timeouts,
- third-party API calls.

Practical guidance:

- Start with Nym's default Mixnet settings.
- Expect a first or long catch-up sync to take longer.
- If a wallet times out once, retry before changing privacy modes.
- Avoid repeatedly disconnecting and reconnecting immediately before a sensitive transaction.
- If your threat model allows it, a less private/faster mode can be used for bulk synchronization, then Mixnet mode can be enabled before sensitive activity. Be aware that contacting the same wallet infrastructure outside the mixnet can reveal additional network metadata.
- If network privacy is the priority, keep the wallet on Mixnet mode and allow more time for synchronization.

## Mobile considerations

On Android and iOS, the system VPN slot is normally the simplest way to protect wallet traffic: connect NymVPN first, then use the wallet.

If another VPN, firewall, or local VPN-based ad blocker already occupies the system VPN interface, the two products may not be able to operate simultaneously. Check the operating system's VPN status before assuming the wallet is protected.

## Threat-model checklist

Before relying on this setup, ask:

- Is the wallet using shielded Zcash addresses where appropriate?
- Is NymVPN visibly connected before the wallet starts network activity?
- Is the wallet excluded by a split-tunneling rule?
- Am I relying on a wallet-specific proxy feature that has actually been documented?
- Am I leaking identity through an exchange account, browser session, or other third-party API?
- Am I prepared for slower sync and occasional timeouts?

## Sources

- Nym Mixnet traffic flow: https://nym.com/docs/network/mixnet-mode/traffic-flow
- NymVPN Mixnet Tuning: https://nym.com/blog/mixnet-tuning
- NymVPN v2026.11 platform/version information: https://nym.com/blog/nymvpn-v2026.11
- NymVPN dApp/wallet mode: https://nym.com/blog/nymvpn-v2026.2
- Nym dApp mode overview: https://nym.com/blog/nymvpn-dapp-mode
- Zodl Tor Protection: https://support.zodl.com/article/17-enabling-tor-protection
- Zodl wallet recovery and Tor sync option: https://support.zodl.com/article/13-recovering-your-zodl-wallet
- Zodl ecosystem page: https://z.cash/ecosystem/zodl-wallet/
- Zkool project (YWallet successor): https://github.com/mladenmarkov/zkool
