# Bitcoin UTXO Management optimization

This guide provides explaining how to utilize them effectively for managing UTXOs (Unspent Transaction Outputs) in a Bitcoin-based application.

## Scripts Description


### `npm run split`

- **Purpose**: Splits a larger UTXO into multiple 546 sats UTXOs. This is useful for transaction privacy and for creating denominations that are easier to split UTXO accurately.
- **Usage**: Run `npm run split` to divide a large UTXO into small ones according to specified criteria.

### `npm run merge`

- **Purpose**: Designed to merge multiple UTXOs into a single UTXO, which can be beneficial for simplifying wallet management and reducing the cost of future transactions.
- **Usage**: Execute `npm run merge` to combine several small UTXOs into one.

### `npm run send`

- **Purpose**: Facilitates the sending of Bitcoins from one address to another. This script likely handles the creation and broadcasting of the transaction to the Bitcoin network.
- **Usage**: Use `npm run send` to specify the details (e.g., recipient's address) and send the UTXO.

## Installation

To set up and run this project locally, follow these steps:

1. **Clone the Repository**: First, clone this repository to your local machine using:
   ```bash
   git clone https://github.com/catlovedev/bitcoin-UTXO-optimization.git
   ```

2. **Install Dependencies**: Navigate into your project directory and install the required dependencies:
   ```bash
   cd bitcoin-UTXO-optimization
   npm install
   ```

3. **Setup Environment Variables**: Copy the `.env.example` file to `.env` and fill in the necessary Bitcoin network details and key management options.
   ```bash
   cp .env.example .env
   ```

4. **Run Scripts**: Use the npm run commands listed above to perform different operations like merging, sending, or splitting UTXOs.

## Dependencies

- **External Modules**:
  - `axios`: For making HTTP requests.
  - `bip32`, `bip39`: For generating Bitcoin wallet addresses from a hierarchical deterministic (HD) path.
  - `bitcoinjs-lib`: A library for Bitcoin protocol functions.
  - `dotenv`: Loads environment variables from a `.env` file.
  - `ecpair`: Represents an elliptic curve pair.
  - `tiny-secp256k1`: For elliptic curve cryptography operations.

- **Development Modules**:
  - `@types/node`: TypeScript type definitions for Node.js.
  - `ts-node`: TypeScript execution environment and REPL for Node.js.
  - `typescript`: The TypeScript compiler.


## Utils Functions

### Interfaces

- **IUtxo:** Interface for Unspent Transaction Outputs which include:
  - **txid:** Transaction ID.
  - **vout:** Output number in the transaction.
  - **value:** Value of the output in satoshis.

### Functions

#### getScriptPubkey(tx: string, address: string): Promise<string>

Fetches the script pubkey associated with a specific output address in a transaction.

**Parameters:**
- `tx`: The transaction ID as a string.
- `address`: The Bitcoin address as a string.

**Returns:** A promise that resolves to the script pubkey as a string.

#### getUtxos(address: string): Promise<IUtxo[]>

Retrieves all UTXOs for a given Bitcoin address.

**Parameters:**
- `address`: The Bitcoin address as a string.

**Returns:** A promise that resolves to an array of UTXOs.

#### pushBTCpmt(rawtx: any): Promise<string>

Pushes a raw Bitcoin transaction to the network.

**Parameters:**
- `rawtx`: The raw transaction data.

**Returns:** A promise that resolves to the transaction ID.

#### postData(url: string, json: any, content_type?: string, apikey?: string): Promise<string | undefined>

General function to send POST requests. Handles error retries related to specific blockchain-related errors.

**Parameters:**
- `url`: The API endpoint.
- `json`: The JSON payload for the post request.
- `content_type`: (Optional) Content type of the request, defaults to `"text/plain"`.
- `apikey`: (Optional) API key for authenticated requests.

**Returns:** A promise that resolves to the response data as a string or undefined in case of specific errors.


## Send UTXO Function

`main(sendToAddress: string)`: this asynchronous function takes a single parameter, `sendToAddress`, which is the Bitcoin address where the funds will be sent.

#### Detailed Workflow

1. **Wallet Initialization**
    - Create an instance of the `Wallet` from the `utils/wallet` module.
    - Log the wallet's address to the console.

2. **UTXO Retrieval**
    - Fetch UTXOs (Unspent Transaction Outputs) associated with the wallet’s address using the `getUtxos` function from `utils/utxo`.
    - Find a suitable UTXO that has a value greater than 10,000 satoshis. If no such UTXO is found, the function throws an error.

3. **PSBT Creation**
    - Initialize a new PSBT (Partially Signed Bitcoin Transaction) specifying the Bitcoin testnet.
    - Add the selected UTXO as an input to the PSBT.
    - Define the transaction outputs:
        - One output sending the intended amount (UTXO value minus initial fee) to `sendToAddress`.
        - Another output to send the remaining balance (initial fee minus calculated transaction fee) back to the wallet's address as a change.

4. **Transaction Fee Calculation**
    - Calculate the transaction fee using the transaction’s virtual size and a predefined testnet fee rate.

5. **Transaction Signing**
    - Sign the PSBT using the wallet’s private keys.

6. **Transaction Extraction and Broadcast**
    - Extract the signed transaction from the PSBT.
    - Convert the transaction to its hexadecimal representation.
    - Broadcast the transaction to the Bitcoin testnet using the `pushBTCpmt` function.

    
## Split UTXO Function

`main()`: this asynchronous function with no parameter

#### Detailed Workflow

1. **Create Wallet Instance**: 
   - Instantiate a new wallet, which will autogenerate a new address and keypair.

2. **Fetch UTXOs**:
   - Retrieve all available UTXOs for the wallet address that have a balance greater than a specified amount (10,000 satoshis in this script).

3. **Transaction Construction**:
   - Create a new `Psbt` (Partially Signed Bitcoin Transaction).
   - Add inputs and outputs to the `Psbt`. Each output sends a fixed amount (546 satoshis in this example) to the wallet address.
   - Calculate and subtract the transaction fees and send the remaining balance back to the wallet address.

4. **Signing the Transaction**:
   - The transaction is signed using the wallet's private key.

5. **Broadcast Transaction**:
   - The transaction, once signed, is converted to hexadecimal format and broadcasted to the network, returning a transaction ID.



## Merge UTXO Function

`main()`: this asynchronous function with no parameter

#### Detailed Workflow

1. **Initialize Wallet**:
   - Creates a new wallet instance which will be used to gather UTXOs and sign the transaction.
   
2. **Fetch UTXOs**:
   - Calls `getUtxos` to retrieve UTXOs for the wallet address.

3. **Prepare Partially Signed Bitcoin Transaction (PSBT)**:
   - Initializes a PSBT object specific to the Bitcoin testnet.
   - Adds inputs for each retrieved UTXO including the necessary details like wallet public key for taproot spend.

4. **Calculate Transaction Fee**:
   - Uses a mock output and fee rate to calculate an appropriate transaction fee based on the virtual size of the transaction.

5. **Add Output**:
   - Adds an output to the transaction, subtracting the calculated fee from the total input value to set the output value.

6. **Sign PSBT**:
   - The wallet signs the PSBT.

7. **Broadcast Transaction**:
   - Extracts the full transaction from PSBT, converts it to hexadecimal, and pushes it to the Bitcoin testnet using `pushBTCpmt`.
   

# Conclusion

As a bitcoin developer, UTXO management is the most basic and important problem. 
If you have any issue for understanding about it, feel free to reach out of me.(https://t.me/CatLoverDev)