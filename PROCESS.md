# Tokeniza Transaction and Process Documentation

This document clarifies the relationship between the Tokeniza off-chain platform and the on-chain smart contracts (`MyToken.sol`, `NFT.sol`).

It is essential to understand that the smart contracts are the final layer of ownership registration. Most business logic, funding, and validation occur on our backend platform *before* any transaction is executed on-chain.

---

## 1. Access Control (Why Mints Are Failing)

Investors observing the contracts on Moonscan may see "failing token mints." This is intentional and a core security feature of our system. Public minting is disabled to protect the integrity of the assets.

* **MyToken.sol (ERC-20):** The `mint` function is protected by `onlyOwner`. Only the Tokeniza backend system (which is the contract 'Owner') can mint new tokens.
* **NFT.sol (ERC-721):** The `mintWithTokenURI` function is protected by `onlyMinter`. Only the Tokeniza backend system (which holds the 'Minter' role) can create new NFTs.

The "failing" transactions are proof that these access controls are working correctly, preventing unauthorized asset creation.

---

## 2. Primary Transaction Flows

### Flow 1: Asset Tokenization (Funding & NFT Mint)

This flow explains how an investor buys a fraction of a Real World Asset (RWA) and receives an NFT representing their ownership.

1.  **Platform (Off-chain):** An investor selects an asset on the Tokeniza platform and initiates the investment.
2.  **Platform (Off-chain):** The investor completes payment (funding) via traditional methods. Our "Payment Processor" (see architecture diagram) confirms receipt.
3.  **Backend (Off-chain):** After successful payment and compliance checks (KYC/AML), the Tokeniza "Token Controller" is triggered.
4.  **Blockchain (On-chain):** The "Token Controller" (acting as the `Minter`) calls the `mintWithTokenURI` function on the `NFT.sol` contract.
5.  **Result:** The NFT is minted directly into the investor's Moonbeam wallet, with the `tokenURI` pointing to metadata that describes the purchased asset.

### Flow 2: Crowdfunding Participation

This flow explains how a user participates in a new offering.

1.  **Platform (Off-chain):** A user completes KYC and registers for an upcoming asset offering (crowdfunding).
2.  **Platform (Off-chain):** During the funding window, the user deposits funds (e.g., USD, BRL) via the platform.
3.  **Backend (Off-chain):** The "Payment Processor" and "Crowdfunding Service" track all contributions.
4.  **Blockchain (On-chain):** Once the offering is successfully closed, the "Token Controller" performs a batch process. It calls `mintWithTokenURI` (for NFTs) or `mint`/`transfer` (for ERC-20s) to distribute the new assets to all participants' wallets.

---

## 3. Secondary Market Flows

### Flow 3: Marketplace Trading (Peer-to-Peer)

This flow explains how an asset is sold on the secondary market.

1.  **Platform (Off-chain):** 'Seller A' lists their NFT (representing an RWA) for sale on the Tokeniza Marketplace.
2.  **Platform (Off-chain):** 'Buyer B' agrees to the price and commits to the purchase. They deposit funds into the platform.
3.  **Backend (Off-chain):** The platform's "Escrow Service" or "Payment Processor" secures the funds from Buyer B.
4.  **Blockchain (On-chain):** The platform, using its authorized "Control Layer," facilitates the atomic swap. This typically involves calling the `transferFrom` function on the `NFT.sol` contract to move the NFT from Seller A to Buyer B.
5.  **Platform (Off-chain):** Once the on-chain transfer is confirmed, the platform releases the off-chain funds to Seller A.

---

## 4. Visualization: Asset Tokenization Flow

```mermaid
sequenceDiagram
    participant Investor
    participant Tokeniza Platform (Front-end)
    participant Tokeniza Backend (Control Layer)
    participant NFT Smart Contract (on-chain)

    Investor->>Tokeniza Platform (Front-end): 1. Initiate Investment
    Tokeniza Platform (Front-end)->>Tokeniza Backend (Control Layer): 2. Submit data (funding, KYC)
    Tokeniza Backend (Control Layer)-->>Tokeniza Backend (Control Layer): 3. Process Payment (off-chain)
    Tokeniza Backend (Control Layer)-->>Tokeniza Backend (Control Layer): 4. Verify Compliance
    alt Transaction Approved
        Tokeniza Backend (Control Layer)->>NFT Smart Contract (on-chain): 5. Call mintWithTokenURI(investorWallet, tokenId, tokenURI)
        NFT Smart Contract (on-chain)-->>Investor: 6. NFT is minted to wallet
    end
