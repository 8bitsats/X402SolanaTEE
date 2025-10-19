# X402SolanaTEE

# 402 API - Solana Edition

Boilerplate for building an API with monetized endpoints using **Solana USDC payments**. All Base/Ethereum dependencies have been completely removed and replaced with native Solana integration.

Live demo at [x402 API in TEE](https://x402.incipient.ltd)

## 📚 Documentation

- **[Quick Start Guide](./QUICKSTART.md)** - Get up and running in 5 minutes
- **[Complete Implementation Guide](./SOLANA_X402_GUIDE.md)** - Full technical documentation
- **[Migration Guide](./SOLANA_MIGRATION.md)** - Migrating from Base/Ethereum to Solana
- **[API Reference](#api-endpoints)** - Endpoint documentation (below)

## Overview

This project provides a production-ready Node.js API server with monetized endpoints using USDC payments on Solana. It includes:

- ⚡ **HTTP 402 Payment Protocol** - Standard-compliant payment required responses
- 💰 **Solana USDC Payments** - Sub-second finality, negligible fees
- 🖼️ **NFT/Asset Queries** - Complete Helius DAS API integration (15 endpoints)
- 🔒 **TEE Support** - Deploy to Phala Cloud for enhanced security
- 🌐 **Web Frontend** - Test interface for API interactions

### Why Solana over Base/Ethereum?

| Feature | Solana | Ethereum/Base |
|---------|--------|---------------|
| **Finality** | ~400ms | 12-30 seconds |
| **Transaction Cost** | <$0.00001 | $1-50 |
| **USDC Support** | Native SPL token | Bridge required |
| **Throughput** | 65,000 TPS | 15-100 TPS |
| **Micropayments** | ✅ Viable | ❌ Too expensive |

The project can be deployed in a Trusted Execution Environment (TEE) on [Phala Cloud](https://cloud.phala.network) to ensure secure and verifiable execution of API calls, protecting sensitive operations and payment transactions.

## API Endpoints

The server exposes the following API endpoints (GET or POST). Each endpoint requires payment via USDC on Solana. Below are detailed descriptions of each endpoint, including how to make calls and expected responses:

### `/text-to-image`

- **Description**: Generates an image based on a text prompt using OpenAI's DALL-E 3 model. This endpoint leverages AI to create visual content from textual descriptions.
- **Payment Required**: Yes ($0.25 USDC on Solana).
- **Method**: GET or POST
- **Query Parameters (for GET)**:
  - `prompt` (string): The text prompt for image generation (defaults to "a beautiful landscape").
- **Request Body (for POST)**:
  - `prompt` (string): The text prompt for image generation.
- **Word Limit**: The prompt is limited to a maximum of 10,000 words.
- **Returns**: JSON object with:
  - `image` (string): URL of the generated image.
  - `prompt` (string): The original prompt used for generation.
- **Example Call** (using curl):
  ```bash
  # First, send USDC payment and get transaction signature
  # Then call API with payment proof
  curl -X POST "http://localhost:4021/text-to-image" \
    -H "Content-Type: application/json" \
    -H "x-solana-signature: YOUR_TX_SIGNATURE" \
    -H "x-solana-payer: YOUR_WALLET_ADDRESS" \
    -d '{"prompt": "a futuristic cityscape"}'
  ```
- **Example Response**:
  ```json
  {
    "image": "https://example.com/generated-image.jpg",
    "prompt": "a futuristic cityscape"
  }
  ```

### `/word-count`

- **Description**: Counts the number of words in a given text. Useful for text analysis or validation tasks.
- **Payment Required**: Yes ($0.01 USDC on Solana).
- **Method**: POST
- **Request Body**:
  - `text` (string): The text for which to count words.
- **Word Limit**: The input text is limited to a maximum of 10,000 words.
- **Returns**: JSON object with:
  - `wordCount` (number): The number of words in the text.
  - `text` (string): The original input text.

### `/sentiment-analysis`

- **Description**: Performs basic sentiment analysis on the provided text, classifying it as positive, negative, or neutral based on keyword scoring.
- **Payment Required**: Yes ($0.05 USDC on Solana).
- **Method**: POST
- **Request Body**:
  - `text` (string): The text to analyze.
- **Word Limit**: The input text is limited to a maximum of 10,000 words.
- **Returns**: JSON object with:
  - `sentiment` (string): The overall sentiment ("positive", "negative", or "neutral").
  - `text` (string): The original input text.
  - `scores` (object): Contains counts of positive and negative words.

### `/test-buy`

- **Description**: A test endpoint for simulating a purchase or transaction. This is useful for developers integrating payment flows.
- **Payment Required**: No, this will use a generated key to pay for the test generation of a red ball.
- **Method**: POST
- **Request Body**: Parameters are hard coded to generate a red ball.
- **Returns**: JSON response indicating the result of the test transaction.

### `GET /`

- **Description**: Serves the `index.html` page from the `public` directory, providing a web interface to interact with the API.
- **Payment Required**: No
- **Method**: GET
- **Returns**: HTML content of the web interface.

## Helius DAS API Endpoints

The server includes comprehensive Solana Digital Asset Standard (DAS) API integration powered by Helius. These endpoints provide rich NFT and asset data, compressed NFT support, and advanced querying capabilities.

### Core Asset Methods

#### `GET /das/asset/:id`
- **Description**: Get detailed information about a specific Solana NFT or digital asset
- **Payment Required**: No
- **Parameters**:
  - `id` (path): Asset ID
  - `displayOptions` (query, optional): JSON string with display options
- **Returns**: Complete asset metadata including ownership, attributes, and collection info
- **Example**:
  ```bash
  curl "http://localhost:4021/das/asset/ASSET_ID_HERE"
  ```

#### `POST /das/assets/batch`
- **Description**: Get multiple assets in a single request (batch query)
- **Payment Required**: No
- **Request Body**:
  - `ids` (array): Array of asset IDs
  - `displayOptions` (object, optional): Display configuration
- **Returns**: Array of asset data
- **Example**:
  ```bash
  curl -X POST "http://localhost:4021/das/assets/batch" \
    -H "Content-Type: application/json" \
    -d '{"ids": ["ASSET_ID_1", "ASSET_ID_2"]}'
  ```

#### `GET /das/asset/:id/proof`
- **Description**: Get cryptographic merkle proof for a compressed NFT (cNFT)
- **Payment Required**: No
- **Parameters**: `id` (path): Compressed NFT asset ID
- **Returns**: Merkle proof data needed for cNFT operations
- **Use Case**: Required for transferring or burning compressed NFTs

#### `POST /das/assets/proof/batch`
- **Description**: Get compression proofs for multiple cNFTs
- **Payment Required**: No
- **Request Body**: `ids` (array): Array of compressed NFT IDs
- **Returns**: Array of proof data

### Asset Query Methods

#### `POST /das/assets/by-owner`
- **Description**: Get all assets owned by a specific wallet address
- **Payment Required**: No
- **Request Body**:
  - `ownerAddress` (string, required): Wallet address
  - `page` (number, optional): Page number (default: 1)
  - `limit` (number, optional): Results per page (default: 100)
  - `displayOptions` (object, optional): Display configuration
- **Returns**: Paginated list of owned assets
- **Example**:
  ```bash
  curl -X POST "http://localhost:4021/das/assets/by-owner" \
    -H "Content-Type: application/json" \
    -d '{"ownerAddress": "WALLET_ADDRESS_HERE", "limit": 50}'
  ```

#### `POST /das/assets/by-authority`
- **Description**: Get all assets by update authority
- **Payment Required**: No
- **Request Body**: Similar to by-owner with `authorityAddress`
- **Returns**: Assets where specified address is the update authority

#### `POST /das/assets/by-creator`
- **Description**: Get all assets created by a specific address
- **Payment Required**: No
- **Request Body**:
  - `creatorAddress` (string, required): Creator wallet address
  - `onlyVerified` (boolean, optional): Only return verified creator assets
  - Pagination options
- **Returns**: Assets created by the specified creator

#### `POST /das/assets/by-group`
- **Description**: Get all assets in a collection or group
- **Payment Required**: No
- **Request Body**:
  - `groupKey` (string, required): Group type (e.g., "collection")
  - `groupValue` (string, required): Collection address
  - Pagination options
- **Returns**: All assets in the specified collection
- **Example**:
  ```bash
  curl -X POST "http://localhost:4021/das/assets/by-group" \
    -H "Content-Type: application/json" \
    -d '{"groupKey": "collection", "groupValue": "COLLECTION_ADDRESS"}'
  ```

### NFT & Token Methods

#### `POST /das/nft/editions`
- **Description**: Get all editions of a master NFT
- **Payment Required**: No
- **Request Body**:
  - `mint` (string, required): Master NFT mint address
  - `page`, `limit` (optional): Pagination
- **Returns**: All edition NFTs of the master

#### `POST /das/token-accounts`
- **Description**: Get token accounts for a specific mint and/or owner
- **Payment Required**: No
- **Request Body**:
  - `owner` (string, required): Wallet address
  - `mint` (string, optional): Token mint address
  - Pagination options
- **Returns**: Token account information

### Search & Utility Methods

#### `POST /das/assets/search`
- **Description**: Advanced search for assets with complex filters
- **Payment Required**: No
- **Request Body**: Extensive filter options including:
  - `ownerAddress`, `creatorAddress`, `authorityAddress`
  - `tokenType`, `compressed`, `frozen`, `burnt`
  - `grouping`, `interface`, sorting, pagination
- **Returns**: Filtered and sorted asset results
- **Example**:
  ```bash
  curl -X POST "http://localhost:4021/das/assets/search" \
    -H "Content-Type: application/json" \
    -d '{
      "ownerAddress": "WALLET_ADDRESS",
      "compressed": true,
      "limit": 20
    }'
  ```

#### `POST /das/priority-fee-estimate`
- **Description**: Get optimal priority fee estimate for Solana transactions
- **Payment Required**: No
- **Request Body**:
  - `transaction` (string): Base64 serialized transaction, OR
  - `accountKeys` (array): Array of account public keys
  - `options` (object, optional): Estimation options
- **Returns**: Priority fee recommendations (min, medium, high)
- **Use Case**: Optimize transaction fees for faster confirmation

### Helper Methods

#### `GET /das/collection/:address`
- **Description**: Get complete collection information and metadata
- **Payment Required**: No
- **Parameters**: `address` (path): Collection address
- **Returns**: Collection details and stats

#### `GET /das/wallet/:address/portfolio`
- **Description**: Get complete wallet portfolio (all NFTs and tokens)
- **Payment Required**: No
- **Parameters**: `address` (path): Wallet address
- **Returns**: Comprehensive portfolio view including fungible tokens

#### `POST /das/verify-ownership`
- **Description**: Verify if a wallet owns a specific asset
- **Payment Required**: No
- **Request Body**:
  - `assetId` (string, required): Asset ID to verify
  - `ownerAddress` (string, required): Expected owner address
- **Returns**: Boolean ownership verification
- **Example**:
  ```bash
  curl -X POST "http://localhost:4021/das/verify-ownership" \
    -H "Content-Type: application/json" \
    -d '{
      "assetId": "ASSET_ID",
      "ownerAddress": "WALLET_ADDRESS"
    }'
  ```

## Trusted Execution Environment (TEE)

This project supports deployment in a Trusted Execution Environment (TEE), as demonstrated in the live demo at [x402 API in TEE](https://x402.incipient.ltd). TEE provides a secure environment for executing code, ensuring that sensitive operations such as payment processing and API key handling are protected from unauthorized access or tampering.

- **How TEE is Used**: When deployed in a TEE, the API server runs within a Trust Domain in an isolated confidential VM. This enclave guarantees that the code and data (like secret salts for key generation, payment transactions, and API responses) are shielded from the host system and external threats.
- **Benefits**: Using TEE enhances trust for users and developers by ensuring that monetized API calls are handled transparently and securely. It prevents potential manipulation of pricing or bypassing payment requirements.
- **Deployment**: The project can be configured to run in TEE platforms like [Phala Cloud](https://cloud.phala.network).

## Project Structure

- **`middleware/`**: Contains Solana payment verification middleware.
- **`handlers/`**: Contains individual handler files for each API endpoint logic.
- **`public/`**: Includes static files for the web frontend (HTML, CSS, JavaScript).
- **`utils/`**: Shared utilities like logging functions used across the project.
- **`server.js`**: Main server file that sets up Express and routes.
- **`testBuy.js`**: Script for testing the Solana payment protocol integration.

## Setup and Deploy on Phala Cloud

1. **Build Docker Image and Publish to DockerHub**:

   ```bash
   # Make sure to log into docker and run docker desktop
   
   # Build docker image
   npx phala docker build
   # Publish docker image
   npx phala docker push
   ```

2. **Environment Variables**:
   Create a `.env` file in the root of the project with the following variables:

   ```env
   # Solana Configuration
   SOLANA_NETWORK=devnet  # or 'mainnet' for production
   WALLET_ADDRESS=your_solana_wallet_address_here
   RPC_URL=https://devnet.helius-rpc.com/?api-key=6b52d42b-5d24-4841-a093-02b0d2cc9fc0

   # Payment Wallet (for testing)
   PRIVATE_KEY=your_base58_private_key_here
   # OR use DSTACK_SECRET_SALT to derive key in TEE
   DSTACK_SECRET_SALT=your_secret_salt_here

   # OpenAI API key for DALL-E image generation
   OPENAI_API_KEY=your_openai_api_key_here

   # API Configuration
   API_URL=http://localhost:4021
   PORT=4021
   ```

3. **Deploy to Phala Cloud**:

   ```bash
   # Make sure to set your API key to access Phala Cloud
   # Replace the image in the docker-compose.yaml file with published image
   
   # Deploy CVM to Phala Cloud
   npx phala cvms create -n x402-app -c docker-compose.yaml -e .env
   ```
   
   The output should provide you a URL to access your CVM in the Phala Cloud Dashboard.

## Test Buy Locally

1. **Install Dependencies**:

   ```bash
   npm install
   ```

2. **Get Test USDC on Devnet**:
   
   You'll need devnet USDC to test payments:
   ```bash
   # Install Solana CLI
   sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
   
   # Create a new keypair (or use existing)
   solana-keygen new --outfile ~/my-wallet.json
   
   # Get devnet SOL for gas
   solana airdrop 2 --url devnet
   
   # Get devnet USDC from https://spl-token-faucet.com/
   ```

3. **Environment Variables**:
   Add the following to your `.env` file for testing:

   ```env
   SOLANA_NETWORK=devnet
   WALLET_ADDRESS=your_solana_wallet_address_here
   PRIVATE_KEY=your_base58_private_key_here
   OPENAI_API_KEY=your_openai_api_key_here
   DSTACK_SECRET_SALT=your_secret_salt_here
   ```

4. **Test Payment Flow**:

   ```bash
   # Start the API server
   npm start
   
   # In another terminal, run a local TEE simulator (optional)
   npx phala simulator start
   
   # Run test buy with generated key from the simulator
   node testBuy.js
   ```

## Payment Flow

The Solana payment system works as follows:

1. **Client makes API request** without payment headers
2. **Server responds with 402 Payment Required**, including:
   - Price in USDC (base units)
   - Receiver wallet address
   - USDC mint address
   - Network (devnet/mainnet)
3. **Client sends USDC payment** on Solana
4. **Client retries request** with payment proof headers:
   - `x-solana-signature`: Transaction signature
   - `x-solana-payer`: Payer wallet address
5. **Server verifies payment** on-chain:
   - Transaction exists and succeeded
   - Correct amount sent to correct receiver
   - USDC token transfer confirmed
6. **Server grants access** to protected endpoint

## Dependencies

- `express`: Web framework for Node.js.
- `@solana/web3.js`: Solana JavaScript SDK.
- `@solana/spl-token`: SPL Token library for USDC operations.
- `@phala/dstack-sdk`: Dstack SDK for TEE functions like generating keys in TEE.
- `dotenv`: Loads environment variables from a `.env` file.
- `openai`: OpenAI Node.js library for accessing the DALL-E API.
- `bs58`: Base58 encoding/decoding for Solana keys.

## Key Improvements over Base

- **Sub-second finality**: Solana confirms transactions in ~400ms vs minutes on Ethereum
- **Negligible fees**: Transaction costs <$0.00001 vs $1-50 on Ethereum
- **Native USDC**: Direct USDC transfers without bridge complexity
- **Better scaling**: Can handle thousands of micropayments per second
- **Simpler integration**: No need for external facilitators or complex L2 setups

---

Learn more about building on Solana at [solana.com](https://solana.com/).
