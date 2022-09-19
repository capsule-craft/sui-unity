# sui-unity-sdk

Connecting Unity game developers to Sui and Origin Byte's NFT ecosystem.

## Features
- Rpc client for direct interaction with the Sui JSON-RPC API https://docs.sui.io/sui-jsonrpc
    - currently exposed: Read API, Event Read API, Transaction Builder Api
- Generate and Restore key pairs with Mnemonics (currently Ed25519 supported), signing transactions
- Currently tested on Windows desktop and WebGL platforms
- Tested with Unity 2021.3.9f1 LTS and greater

## Getting Started

### To try everything out: Download this repository and open with Unity
### To import into your project:
    - TODO UPN
    - TODO  .unitypackage
    - TODO gihub url

### Export a unitypackage from the SDK in this repository
1. Click the RMB on the SuiUnitySDK folder to get a context menu
2. Click Export Package on the context menu
3. Uncheck the *Include dependencies* checkbox not to export non-sdk files
4. Click the Export button
5. Write a name of the package
6. Click the Save button
7. The unitypackage is saved on your disk

### Import the SDK unitypackage to an Unity project

1. Create an Unity project
2. Drag and drop the unitypackage to the Project window
3. As soon as the Import Windows pop up, just click the Import button
4. The SDK will be imported

## Usage Samples

All samples can be found in SuiUnitySDK/Samples

### Create New Wallet and Import Wallet

![Alt text](/imgs/create_new_wallet.png "Create New Wallet")

You can click on Create New Wallet or Import Wallet buttons to initialize the currently active Wallet.
PlayerPrefs is used as a keystore at the moment, and it saves the last active wallet and will be loaded from there on the next restart.
Now you are ready to execute transactions that require signature.

```csharp
 var mnemo = Mnemonics.GenerateMnemonic();
 ...
 Mnemonics.GetKeypairFromMnemonic(mnemo).PublicKeySuiAddress;
```

### RPC Read API
![Alt text](/imgs/read_api.png "Read API test")

Enter any address and see the results as formatted JSON.

```csharp
    var rpcClient = new UnityWebRequestRpcClient(SuiConstants.DEVNET_ENDPOINT);
    var suiJsonRpcApi = new SuiJsonRpcApiClient(rpcClient);
    var ownedObjectsResult = await suiJsonRpcApi.GetObjectsOwnedByAddressAsync(Input.text);
    Ouput.text = JsonConvert.SerializeObject(ownedObjectsResult.Result, Formatting.Indented);
```

### RPC Move call and execute transaction samples

![Alt text](/imgs/transactions.png "Transactions")

This sample code calls a function in a published move package that has a ```counter``` module with an ```increment``` function. It modifies a shared object, incrementing the counter by 1. 

See move logic here: https://github.com/MystenLabs/sui/blob/main/sui_programmability/examples/basics/sources/counter.move

```csharp
    var rpcClient = new UnityWebRequestRpcClient(SuiConstants.DEVNET_ENDPOINT);
    var suiJsonRpcApi = new SuiJsonRpcApiClient(rpcClient);

    var signer = SuiWallet.Instance.GetActiveAddress();
    var packageObjectId = "0xa21da7987c2b75870ddb4d638600f9af950b64c6";
    var module = "counter";
    var function = "increment";
    var typeArgs = System.Array.Empty<string>();
    var args = new object[] { SharedCounterObjectId };
    var gasObjectId = GasObjectIdInput.text;
    var rpcResult = await suiJsonRpcApi.MoveCallAsync(signer, packageObjectId, module, function, typeArgs, args, gasObjectId, 2000);
    var keyPair = SuiWallet.Instance.GetActiveKeyPair();

    var txBytes = rpcResult.Result.TxBytes;
    var signature = keyPair.Sign(rpcResult.Result.TxBytes);
    var pkBase64 = keyPair.PublicKeyBase64;

    await suiJsonRpcApi.ExecuteTransactionAsync(txBytes, SuiSignatureScheme.ED25519, signature, pkBase64);
    ...
```

## Dependencies

Every dependency with source code can be found in ./Assets/AldrinLabsSDK/Plugins.

Nuget packages are in ./Assets/AldrinLabsSDK/NuGetPackages

- `suinet`, our internal C#/.NET Sui library. It will be regularly updated as features are added to the core library.
- `Newtonsoft.Json` (included in Unity by default)
- `Chaos.NaCl.Standard`
- `Portable.BouncyCastle`

## Roadmap
- Mobile platform support (iOS, Android)
- WalletConnect
- Streaming RPC client, Event subscription
- Secp256k1 keypair support
- More samples
- Origin-Byte NFT ecosystem access from Unity
- Full Rust & Typescript SDK parity
